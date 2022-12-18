---
layout: code
title: 4D Bivector and Rotor Code (C#)
---

{% highlight cs %}
{% raw %}

/*
 * Based on a c++ template by Marc ten Bosch
 * https://marctenbosch.com/news/2011/05/4d-rotations-and-the-4d-equivalent-of-quaternions/
 * 
 * Rotor4 was founded on Marc ten Bosch's Rotor3
 * https://marctenbosch.com/quaternions/code.htm
 *
 * Rotor equation derivations are explained in https://joesubbi.github.io/2022/04/04/4D-Rotation/
 */

public class Bivector4
{
    public float bxy = 0;
    public float bxz = 0;
    public float byz = 0;
    public float bxw = 0;
    public float byw = 0;
    public float bzw = 0;

    public Bivector4(float bxy, float bxz, float byz,
                  float bxw, float byw, float bzw)
    {
        this.bxy = bxy;
        this.bxz = bxz;
        this.byz = byz;
        this.bxw = bxw;
        this.byw = byw;
        this.bzw = bzw;
    }

    public static Bivector4 Wedge(Vector4 u, Vector4 v)
    {
        Bivector4 b = new Bivector4(
                u.x * v.y - u.y * v.x, //XY
                u.x * v.z - u.z * v.x, //XZ
                u.y * v.z - u.z * v.y, //YZ
                u.x * v.w - u.w * v.x, //XW
                u.y * v.w - u.w * v.y, //YW
                u.z * v.w - u.w * v.z  //ZW
            );
        return b;
    }
}

public class Rotor4
{
    // scalar part
    public float a = 1;

    // bivector part
    public float bxy = 0;
    public float bxz = 0;
    public float byz = 0;
    public float bxw = 0;
    public float byw = 0;
    public float bzw = 0;

    // pseudo-vector part
    public float pxyzw = 0;

    private const float epsilon = 1e-6f;

    private static Rotor4 identity = new Rotor4(1, 0, 0, 0, 0, 0, 0, 0);

    // ---- CONSTRUCTORS ----

    public Rotor4(float a, float bxy, float bxz, float byz,
                           float bxw, float byw, float bzw, float bxyzw)
    {
        this.a = a;
        this.bxy = bxy;
        this.bxz = bxz;
        this.byz = byz;
        this.bxw = bxw;
        this.byw = byw;
        this.bzw = bzw;
        this.pxyzw = bxyzw;
    }

    public Rotor4(Bivector4 bv, float angle)
    {
        float sina = Mathf.Sin(angle / 2);
        a = Mathf.Cos(angle / 2);
        bxy = -sina * bv.bxy;
        bxz = -sina * bv.bxz;
        byz = -sina * bv.byz;
        bxw = -sina * bv.bxw;
        byw = -sina * bv.byw;
        bzw = -sina * bv.bzw;
        pxyzw = 0;

        Normalise();
    }

    public static Rotor4 Identity() { return identity; }

    // ---- OPERATORS ----

    // Rotor4-Rotor4 Product
    public static Rotor4 operator *(Rotor4 a, Rotor4 b)
    {
        float e     = -a.bxw * b.bxw   - a.bxy * b.bxy   - a.bxz * b.bxz   - a.byw * b.byw   - a.byz * b.byz   - a.bzw * b.bzw   + a.pxyzw * b.pxyzw + a.a * b.a;
        float exy   = -a.bxw * b.byw   + a.bxy * b.a     - a.bxz * b.byz   + a.byw * b.bxw   + a.byz * b.bxz   - a.bzw * b.pxyzw - a.pxyzw * b.bzw   + a.a * b.bxy;
        float exz   = -a.bxw * b.bzw   + a.bxy * b.byz   + a.bxz * b.a     + a.byw * b.pxyzw - a.byz * b.bxy   + a.bzw * b.bxw   + a.pxyzw * b.byw   + a.a * b.bxz;
        float exw   =  a.bxw * b.a     + a.bxy * b.byw   + a.bxz * b.bzw   - a.byw * b.bxy   - a.byz * b.pxyzw - a.bzw * b.bxz   - a.pxyzw * b.byz   + a.a * b.bxw;
        float eyz   = -a.bxw * b.pxyzw - a.bxy * b.bxz   + a.bxz * b.bxy   - a.byw * b.bzw   + a.byz * b.a     + a.bzw * b.byw   - a.pxyzw * b.bxw   + a.a * b.byz;
        float eyw   =  a.bxw * b.bxy   - a.bxy * b.bxw   + a.bxz * b.pxyzw + a.byw * b.a     + a.byz * b.bzw   - a.bzw * b.byz   + a.pxyzw * b.bxz   + a.a * b.byw;
        float ezw   =  a.bxw * b.bxz   - a.bxy * b.pxyzw - a.bxz * b.bxw   + a.byw * b.byz   - a.byz * b.byw   + a.bzw * b.a     - a.pxyzw * b.bxy   + a.a * b.bzw;
        float exyzw =  a.bxw * b.byz   + a.bxy * b.bzw   - a.bxz * b.byw   - a.byw * b.bxz   + a.byz * b.bxw   + a.bzw * b.bxy   + a.pxyzw * b.a     + a.a * b.pxyzw;

        return new Rotor4(e, exy, exz, eyz, exw, eyw, ezw, exyzw);
    }

    // Rotor4-Vector4 Product (Rotation)
    public static Vector4 operator *(Rotor4 r, Vector4 a)
    {
        return r.Rotate(a);
    }

    public static Vector4 operator *(Vector4 a, Rotor4 r)
    {
        return r.Rotate(a);
    }

    // Rotate a Vector with a Rotor
    public Vector4 Rotate(Vector4 a)
    {
        float s = this.a;
        float s2 = s * s;
        float bxy2 = bxy * bxy;
        float bxz2 = bxz * bxz;
        float bxw2 = bxw * bxw;
        float byz2 = byz * byz;
        float byw2 = byw * byw;
        float bzw2 = bzw * bzw;
        float bxyzw2 = pxyzw * pxyzw;

        Vector4 r;

        r.x = (
              2 * a.w * bxw * s
            + 2 * a.w * bxy * byw
            + 2 * a.w * bxz * bzw
            + 2 * a.w * byz * pxyzw
            - a.x * bxw2
            - a.x * bxy2
            - a.x * bxz2
            + a.x * byw2
            + a.x * byz2
            + a.x * bzw2
            - a.x * bxyzw2
            + a.x * s2
            - 2 * a.y * bxw * byw
            + 2 * a.y * bxy * s
            - 2 * a.y * bxz * byz
            + 2 * a.y * bzw * pxyzw
            - 2 * a.z * bxw * bzw
            + 2 * a.z * bxy * byz
            + 2 * a.z * bxz * s
            - 2 * a.z * byw * pxyzw
        );
        r.y = (
            - 2 * a.w * bxw * bxy
            - 2 * a.w * bxz * pxyzw
            + 2 * a.w * byw * s
            + 2 * a.w * byz * bzw
            - 2 * a.x * bxw * byw
            - 2 * a.x * bxy * s
            - 2 * a.x * bxz * byz
            - 2 * a.x * bzw * pxyzw
            + a.y * bxw2
            - a.y * bxy2
            + a.y * bxz2
            - a.y * byw2
            - a.y * byz2
            + a.y * bzw2
            - a.y * bxyzw2
            + a.y * s2
            + 2 * a.z * bxw * pxyzw
            - 2 * a.z * bxy * bxz
            - 2 * a.z * byw * bzw
            + 2 * a.z * byz * s
        );
        r.z = (
            - 2 * a.w * bxw * bxz
            + 2 * a.w * bxy * pxyzw
            - 2 * a.w * byw * byz
            + 2 * a.w * bzw * s
            - 2 * a.x * bxw * bzw
            + 2 * a.x * bxy * byz
            - 2 * a.x * bxz * s
            + 2 * a.x * byw * pxyzw
            - 2 * a.y * bxw * pxyzw
            - 2 * a.y * bxy * bxz
            - 2 * a.y * byw * bzw
            - 2 * a.y * byz * s
            + a.z * bxw2
            + a.z * bxy2
            - a.z * bxz2
            + a.z * byw2
            - a.z * byz2
            - a.z * bzw2
            - a.z * bxyzw2
            + a.z * s2

        );
        r.w = (
            - a.w * bxw2
            + a.w * bxy2
            + a.w * bxz2
            - a.w * byw2
            + a.w * byz2
            - a.w * bzw2
            - a.w * bxyzw2
            + a.w * s2
            - 2 * a.x * bxw * s
            + 2 * a.x * bxy * byw
            + 2 * a.x * bxz * bzw
            - 2 * a.x * byz * pxyzw
            - 2 * a.y * bxw * bxy
            + 2 * a.y * bxz * pxyzw
            - 2 * a.y * byw * s
            + 2 * a.y * byz * bzw
            - 2 * a.z * bxw * bxz
            - 2 * a.z * bxy * pxyzw
            - 2 * a.z * byw * byz
            - 2 * a.z * bzw * s
        );

        return r;
    }

    // Rotate one Rotor to another
    public Rotor4 Rotate(Rotor4 r)
    {
        return this * r * this.Reverse();
    }

    // Spherical Linear Interpolation between two Rotors (MOSTLY WORKS - NOT PERFECT - FIX IN PROGRESS)
    public static Rotor4 SLerp(Rotor4 from, Rotor4 to, float ratio, bool normalise = true)
    {
        // The following SLerp is from:
        // https://referencesource.microsoft.com/#System.Numerics/System/Numerics/Quaternion.cs

        float t = ratio;
 
        float sq(float x) => x * x;

        Rotor4 difference = RotorFromTo(from, to);
        float cosOmega = Mathf.Sqrt(sq(difference.a) + sq(difference.pxyzw));

        bool flip = false;
 
        if (cosOmega < 0.0f)
        {
            flip = true;
            cosOmega = -cosOmega;
        }
 
        float s1, s2;
 
        if (cosOmega > (1.0f - epsilon))
        {
            // Too close, do straight linear interpolation.
            s1 = 1.0f - t;
            s2 = (flip) ? -t : t;
        }
        else
        {
            float omega = (float)Mathf.Acos(cosOmega);
            float invSinOmega = (float)(1 / Mathf.Sin(omega));
 
            s1 = (float)Mathf.Sin((1.0f - t) * omega) * invSinOmega;
            s2 = (flip)
                ? (float)-Mathf.Sin(t * omega) * invSinOmega
                : (float)Mathf.Sin(t * omega) * invSinOmega;
        }

        Rotor4 toReturn = new Rotor4(s1 * from.a + s2 * to.a,
                                     s1 * from.bxy + s2 * to.bxy,
                                     s1 * from.bxz + s2 * to.bxz,
                                     s1 * from.byz + s2 * to.byz,
                                     s1 * from.bxw + s2 * to.bxw,
                                     s1 * from.byw + s2 * to.byw,
                                     s1 * from.bzw + s2 * to.bzw,
                                     s1 * from.pxyzw + s2 * to.pxyzw);
        if (normalise)
            toReturn.Normalise();

        return toReturn;
        

        // Get the angle between rotors
        // create a rotor, but with the angle scaled by t

        //Rotor4 toReturn = RotorFromTo(from, to);
    }

    // ---- ROTOR OPERATIONS ----

    // Conjugate
    public Rotor4 Reverse()
    {
        return new Rotor4(a, -bxy, -bxz, -byz, -bxw, -byw, -bzw, pxyzw);
    }

    // Length Squared
    private float LengthSquared()
    {
        float sq(float x) => x * x;

        return sq(a) + sq(bxy) + sq(bxz) + sq(byz) +
                       sq(bxw) + sq(byw) + sq(bzw) + sq(pxyzw);
    }

    // Length
    private float Length()
    {
        return Mathf.Sqrt(LengthSquared());
    }

    // Normalise this Rotor
    public void Normalise()
    {
        float l = Length();
        a /= l;
        bxy /= l;
        bxz /= l;
        byz /= l;
        bxw /= l;
        byw /= l;
        bzw /= l;
        pxyzw /= l;
    }

    // Normalised copy of a Rotor
    public Rotor4 Normal()
    {
        Rotor4 r = this;
        r.Normalise();
        return r;
    }

    // The Rotor to rotate one Rotor to another
    public static Rotor4 RotorFromTo(Rotor4 a, Rotor4 b)
    {
        Rotor4 difference = b * a.Reverse();
        difference.Normalise();
        return difference;
    }

    // The Angle between two Rotors (radians)
    public static float AngleFromTo(Rotor4 a, Rotor4 b)
    {
        float sq(float x) => x * x;

        Rotor4 difference = RotorFromTo(a, b);
        return Mathf.Acos( Mathf.Sqrt(sq(difference.a) + sq(difference.pxyzw)) ) * 2;
    }

    // The Rotor to get from a to b, and the angle between rotor
    public static (Rotor4, float) Difference(Rotor4 a, Rotor4 b)
    {
        Rotor4 difference = RotorFromTo(a, b);

        float angle = AngleFromTo(a, b);

        return (difference, angle);
    }

    // Dot Product of two Rotors
    public static Rotor4 Dot(Rotor4 a, Rotor4 b)
    {
        float e = -a.bxw * b.bxw - a.bxy * b.bxy - a.bxz * b.bxz - a.byw * b.byw - a.byz * b.byz - a.bzw * b.bzw + a.pxyzw - b.pxyzw;
        float exy = -a.bzw * b.pxyzw - a.pxyzw * b.bzw;
        float exz =  a.byw * b.pxyzw + a.pxyzw * b.byw;
        float exw = -a.byz * b.pxyzw + a.pxyzw * b.byz;
        float eyz = -a.bxw * b.pxyzw + a.pxyzw * b.bxw;
        float eyw =  a.bxz * b.pxyzw + a.pxyzw * b.bxz;
        float ezw = -a.bxy * b.pxyzw + a.pxyzw * b.bxy;
        float exyzw = 0;

        return new Rotor4(e, exy, exz, eyz, exw, eyw, ezw, exyzw);
    }

    // Geometric Product
    public static Rotor4 GeoProd(Vector4 a, Vector4 b)
    {
        Bivector4 bv = Bivector4.Wedge(a, b);
        return new Rotor4(Vector4.Dot(a, b), bv.bxy, bv.bxz, bv.byz,
                                             bv.bxw, bv.byw, bv.bzw, 0);
    }

    // ---- EQUALITY OPERATIONS ----

    public static bool ExactlyEqual(Rotor4 a, Rotor4 b)
    {        
        if (a.a   == b.a   &&
            a.byz == b.byz &&
            a.bxz == b.bxz &&
            a.bxy == b.bxy &&
            a.bxw == b.bxw &&
            a.byw == b.byw &&
            a.bzw == b.bzw)
            return true;
        return false;
    }

    public static bool ApproxEqual(Rotor4 a, Rotor4 b, float e = 0.005f)
    {
        float angle = AngleFromTo(a, b);
        return angle < e;
    }

    public static bool operator ==(Rotor4 lhs, Rotor4 rhs)
    {
        return ApproxEqual(lhs, rhs);
    }

    public static bool operator !=(Rotor4 lhs, Rotor4 rhs)
    {
        // Returns true in the presence of NaN values.
        return !(lhs == rhs);
    }

    public bool Equals(Rotor4 rotor) { return this == rotor; }

    public override bool Equals(object obj) => Equals(obj as Rotor4);

    public override int GetHashCode()
    {
        return HashCode.Combine(a, bxy, bxz, byz, bxw, byw, bzw, pxyzw);
    }

    // ---- Utility Functions ----

    public override string ToString()
    {
        return a + " + " + bxy + " + " + bxz + " + " + byz + " + " + bxw + " + " + byw + " + " + bzw + " + " + pxyzw;
    }

    public float[] ToArray() { return new float[] { a, bxy, bxz, byz, bxw, byw, bzw, pxyzw }; }
}

{% endraw %}
{% endhighlight %}
