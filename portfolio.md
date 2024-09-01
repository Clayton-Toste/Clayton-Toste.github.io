---
layout: page
title: Portfolio
permalink: /portfolio/
classes: wide
---

<style type="text/css">
.wrapper {
  max-width: 90%;
}
.column1 {
  width: 25%;
  margin-right: 75%;
  position: fixed;
}
  
.column2 {
  width: 75%;
  margin-left: 25%;
}
</style>

<div class="column1">
<h2>Table Contents</h2>
<ul>
  <li><a href="splinerail">SplineRail</a></li>
</ul>  
</div>

<div class="column2">
<p align=middle>
  <h2 id="splinerail" align=middle><i>Spline Rail</i></h2>
</p>

<p align=middle>
  <video controls width="50%">
    <source src="../splinerail.webm" type="video/webm" />
  </video>
</p>

<p align=middle>
Spline Rail is a scirpt I made for a game I'm currently developing. It calculates a spline from control points and renders it to a 3D mesh. It also provides functions for fetching information on the rail to allow for riding. Additionally It allows for realtime editing through Unity's Editor system. The script also features caching spline information by using hashes to check for changes.
</p>


<details>
  <summary> Code </summary>
  <pre>
    <code>
using System;
using System.Collections.Generic;
using System.Linq;
using Unity.Mathematics;
using UnityEditor;
using UnityEngine;

/// &#x3C;summary&#x3E;
/// Represents a grindable rail along a Catmull-Rom spline.
/// &#x3C;/summary&#x3E;
[ExecuteInEditMode, RequireComponent(typeof(MeshFilter)), RequireComponent(typeof(MeshCollider))]
public class SplineRail : MonoBehaviour, IRail
{
    struct Subcurve
    {
        Vector3[] factors;

        /// &#x3C;summary&#x3E;
        /// Constructs a subcurve from 4 control points;
        /// &#x3C;/summary&#x3E;
        /// &#x3C;param name=&#x22;controlPoint1&#x22;&#x3E; First control point. &#x3C;/param&#x3E;
        /// &#x3C;param name=&#x22;controlPoint2&#x22;&#x3E; Second control point. &#x3C;/param&#x3E;
        /// &#x3C;param name=&#x22;controlPoint3&#x22;&#x3E; Third control point. &#x3C;/param&#x3E;
        /// &#x3C;param name=&#x22;controlPoint4&#x22;&#x3E; Fourth control point. &#x3C;/param&#x3E;
        public Subcurve(Vector3 controlPoint1, Vector3 controlPoint2, Vector3 controlPoint3, Vector3 controlPoint4)
        {
            factors = new Vector3[4];
            factors[0] = controlPoint2;
            factors[1] = (controlPoint3 - controlPoint1) / 2.0f;
            factors[2] = controlPoint1 + 2.0f * controlPoint3 - (5.0f * controlPoint2 + controlPoint4) / 2.0f;
            factors[3] = (3 * controlPoint2 + controlPoint4 - controlPoint1 - 3.0f * controlPoint3) / 2.0f;
        }

        /// &#x3C;summary&#x3E;
        /// Evaluates the curve at a collection of t values.
        /// &#x3C;/summary&#x3E;
        /// &#x3C;param name=&#x22;t1&#x22;&#x3E; The first t value. &#x3C;/param&#x3E;
        /// &#x3C;param name=&#x22;t2&#x22;&#x3E; The second t value. &#x3C;/param&#x3E;
        /// &#x3C;param name=&#x22;t3&#x22;&#x3E; The third t value.&#x3C;/param&#x3E;
        /// &#x3C;param name=&#x22;t4&#x22;&#x3E; The fourth t value.&#x3C;/param&#x3E;
        /// &#x3C;returns&#x3E; The value along the curve. &#x3C;/returns&#x3E;
        public readonly Vector3 Evaluate(float t1, float t2, float t3, float t4)
        {
            return t1 * factors[0] + t2 * factors[1] + t3 * factors[2] + t4 * factors[3];
        }
    }

    struct Subline
    {
        Vector3 start, path;
        float startT, pathT;

        /// &#x3C;summary&#x3E;
        /// Constructs a subline from a start and a end.
        /// &#x3C;/summary&#x3E;
        /// &#x3C;param name=&#x22;start&#x22;&#x3E; The start of the curve. &#x3C;/param&#x3E;
        /// &#x3C;param name=&#x22;end&#x22;&#x3E; The end of the curve. &#x3C;/param&#x3E;
        public Subline(Vector3 start, float startT, Vector3 end, float endT)
        {
            this.start = start;
            path = end - start;
            this.startT = startT;
            pathT = endT - startT;
        }

        /// &#x3C;summary&#x3E;
        /// Constructs a subline from a start and a end.
        /// &#x3C;/summary&#x3E;
        /// &#x3C;param name=&#x22;start&#x22;&#x3E; The start of the curve. &#x3C;/param&#x3E;
        /// &#x3C;param name=&#x22;end&#x22;&#x3E; The end of the curve. &#x3C;/param&#x3E;
        public Subline((Vector3 point, float t) start, (Vector3 point, float t) end)
        {
            (this.start, startT) = start;
            path = end.point - start.point;
            pathT = end.t - start.t;
        }

        /// &#x3C;summary&#x3E;
        /// Evaluates the line at a t value.
        /// &#x3C;/summary&#x3E;
        /// &#x3C;param name=&#x22;t&#x22;&#x3E; The t value. &#x3C;/param&#x3E;
        /// &#x3C;returns&#x3E; The value along the curve. &#x3C;/returns&#x3E;
        public readonly (Vector3 point, float t) Evaluate(float t)
        {
            return (start + t * path, startT + t * pathT);
        }

        /// &#x3C;summary&#x3E;
        /// Find the closest global t to a position.
        /// &#x3C;/summary&#x3E;
        /// &#x3C;param name=&#x22;position&#x22;&#x3E; the position to the closest t to. &#x3C;/param&#x3E;
        /// &#x3C;returns&#x3E; The tuple of the closest global t and the distance to that t. &#x3C;/returns&#x3E;
        public readonly (float, float) ClosestGlobalT(Vector3 position)
        {
            (Vector3 point, float t) = Evaluate(Mathf.Clamp(Vector3.Dot(position - start, path) / path.sqrMagnitude, 0.0f, 1.0f));
            return (t, (position - point).magnitude);
        }
    }

    /// &#x3C;summary&#x3E;
    /// The size of the step between joints.
    /// &#x3C;/summary&#x3E;
    [Header(&#x22;Configuration&#x22;)]
    public float stepLength = 0.5f;
    int stepSizeHash;

    /// &#x3C;summary&#x3E;
    /// How many substeps to take between joints.
    /// &#x3C;/summary&#x3E;
    public int precision = 10;
    int precisionHash;

    /// &#x3C;summary&#x3E;
    /// The Catmull-Rom spline as control points.
    /// &#x3C;/summary&#x3E;
    public Vector3[] controlPoints = new Vector3[]
    {
        new (-1.0f, 0.0f),
        new (1.0f, 0.0f)
    };
    int controlPointsHash;

    /// &#x3C;summary&#x3E;
    /// Reference joint along the rail.
    /// &#x3C;/summary&#x3E;
    public Vector2[] joint = new Vector2[]
    {
        new (0.3f, 0.3f),
        new (-0.3f, 0.3f),
        new (-0.3f, -0.3f),
        new (0.3f, -0.3f),
    };
    int jointHash;

    Bounds jointBounds;


    readonly List&#x3C;Subcurve&#x3E; spline = new();
    readonly List&#x3C;Subline&#x3E; lines = new();

    void Update()
    {
      CheckForUpdates()
    }
    
    /// &#x3C;summary&#x3E;
    /// Forces the rail to check for updates in it's structure.
    /// &#x3C;/summary&#x3E;
    public void CheckForUpdates()
    {
        int controlPointsHash = 0,
        stepSizeHash = stepLength.GetHashCode(),
        precisionHash = precision.GetHashCode(),
        jointHash = 0;

        foreach (Vector2 point in controlPoints)
        {
            controlPointsHash ^= point.GetHashCode();
        }
        foreach (Vector2 point in joint)
        {
            jointHash ^= point.GetHashCode();
        }

        if (this.jointHash != jointHash)
        {
            PreprocessJoint();
        }
        if (this.controlPointsHash != controlPointsHash || this.stepSizeHash != stepSizeHash || this.precisionHash != precisionHash)
        {
            PreprocessSpline();
        }
        if (this.controlPointsHash != controlPointsHash || this.stepSizeHash != stepSizeHash || this.precisionHash != precisionHash || this.jointHash != jointHash)
        {
            UpdateMesh();
        }

        if (this.controlPointsHash != controlPointsHash)
        {
            this.controlPointsHash = controlPointsHash;
        }
        if (this.stepSizeHash != stepSizeHash)
        {
            this.stepSizeHash = stepSizeHash;
        }
        if (this.precisionHash != precisionHash)
        {
            this.precisionHash = precisionHash;
        }
        if (this.jointHash != jointHash)
        {
            this.jointHash = jointHash;
        }
    }

    void PreprocessSpline()
    {
        if (precision == 0 || stepLength == 0)
        {
            throw new ArgumentException("Precision and stepLength must not be 0.");
        }

        // Construct subcurves from control points
        spline.Clear();

        for (int i = 0; i + 1 < controlPoints.Length; i++)
        {
            spline.Add(new(GetControlPoint(i - 1), GetControlPoint(i), GetControlPoint(i + 1), GetControlPoint(i + 2)));
        }

        // Construct sublines between evenly spaced intervals.
        lines.Clear();

        float t = 0.0f;

        // Increase t by enough to move stepLength along the curve
        for (int i = 0; i < precision; i++)
        {
            t += stepLength / (precision * SplineVelocity(t).magnitude);
            t = Mathf.Clamp(t, 0.0f, 1.0f);
        }

        lines.Add(new(SplineDisplacement(0.0f), 0.0f, SplineDisplacement(t), t));

        while (t < 1.0f)
        {
            // Increase t by enough to move stepLength along the curve
            for (int i = 0; i < precision; i++)
            {
                t += stepLength / (precision * SplineVelocity(t).magnitude);
                t = Mathf.Clamp(t, 0.0f, 1.0f);
            }

            lines.Add(new(lines[^1].Evaluate(1.0f), (SplineDisplacement(t), t)));
        }
    }

    void PreprocessJoint()
    {
        // Find bounding box of joint
        jointBounds = new();
        foreach (Vector3 point in joint)
        {
            jointBounds.Encapsulate(point);
        }
    }

    /// &#x3C;summary&#x3E;
    /// Updates &#x3C;c&#x3E; mesh &#x3C;/c&#x3E; to reflect &#x3C;c&#x3E; controlPoints &#x3C;/c&#x3E;.
    /// &#x3C;/summary&#x3E;
    public void UpdateMesh()
    {
        if (lines.Count == 0)
        {
            return;
        }

        List<Vector3> vertices = new();
        List<int> triangles = new();
        List<Vector2> uv = new();

        foreach (Subline line in lines)
        {
            float t = line.Evaluate(0.0f).t;
            Quaternion direction = SplineRotation(t);
            Vector3 displacement = SplineDisplacement(t);
            // Add vertices and UV for joint
            vertices.Add(direction * (Vector3)joint[0] + displacement);
            uv.Add(new Vector2(0.0f, 0.0f));
            for (int i = 1; i < joint.Count(); i++)
            {
                vertices.Add(direction * (Vector3)joint[i] + displacement);
                uv.Add(new Vector2(0.0f, 1.0f));
                vertices.Add(direction * (Vector3)joint[i] + displacement);
                uv.Add(new Vector2(0.0f, 0.0f));
            }
            vertices.Add(direction * (Vector3)joint[0] + displacement);
            uv.Add(new Vector2(0.0f, 1.0f));

            // Add connecting faces
            for (int i = vertices.Count; i < vertices.Count + 2 * joint.Count(); i += 2)
            {
                triangles.Add(i - 8);
                triangles.Add(i - 7);
                triangles.Add(i + 1);

                triangles.Add(i + 1);
                triangles.Add(i);
                triangles.Add(i - 8);
            }

            t = line.Evaluate(1.0f).t;
            direction = SplineRotation(t);
            displacement = SplineDisplacement(t);
            // Add vertices and UV for joint
            vertices.Add(direction * (Vector3)joint[0] + displacement);
            uv.Add(new Vector2(1.0f, 0.0f));
            for (int i = 1; i < joint.Count(); i++)
            {
                vertices.Add(direction * (Vector3)joint[i] + displacement);
                uv.Add(new Vector2(1.0f, 1.0f));
                vertices.Add(direction * (Vector3)joint[i] + displacement);
                uv.Add(new Vector2(1.0f, 0.0f));
            }
            vertices.Add(direction * (Vector3)joint[0] + displacement);
            uv.Add(new Vector2(1.0f, 1.0f));
        }

        Mesh mesh = new() { vertices = vertices.ToArray(), triangles = triangles.ToArray(), uv = uv.ToArray()};
        mesh.Optimize();
        mesh.RecalculateNormals();
        TryGetComponent(out MeshFilter filter);
        filter.mesh = mesh;
        TryGetComponent(out MeshCollider collider);
        collider.sharedMesh = mesh;
    }

    /// &#x3C;summary&#x3E;
    /// Retrieves the &#x3C;paramref name=&#x22;i&#x22;/&#x3E;th control point along the spline. Will extrapolate one point ahead or behind.
    /// &#x3C;/summary&#x3E;
    /// &#x3C;param name=&#x22;i&#x22;&#x3E; Index of the control point to retrieve. &#x3C;/param&#x3E;
    /// &#x3C;returns&#x3E; &#x3C;paramref name=&#x22;i&#x22;/&#x3E;th control point. &#x3C;/returns&#x3E;
    /// &#x3C;exception cref=&#x22;IndexOutOfRangeException&#x22;&#x3E; Thrown when index &#x3C;paramref name=&#x22;i&#x22;/&#x3E; is out of range [-1, &#x3C;c&#x3E;controlPoints.Count&#x3C;/c&#x3E;]. &#x3C;/exception&#x3E;
    /// &#x3C;exception cref=&#x22;IndexOutOfRangeException&#x22;&#x3E; Thrown when there are too few points to extrapolate (less than 2) and the index is on the bounds. &#x3C;/exception&#x3E;
    public Vector3 GetControlPoint(int i)
    {
        if (i &#x3C; -1 || i &#x3E; controlPoints.Length)
        {
            throw new IndexOutOfRangeException($&#x22;Index {i} out of bounds, should be in [-1 and {controlPoints.Length}].&#x22;);
        }

        if (i == -1)
        {
            if (controlPoints.Length &#x3C; 2)
            {
                throw new IndexOutOfRangeException($&#x22;Tried to extrapolate with too few points.&#x22;);
            }
            return 2 * controlPoints[0] - controlPoints[1];
        }


        if (i == controlPoints.Length)
        {
            if (controlPoints.Length &#x3C; 2)
            {
                throw new ArgumentException($&#x22;Tried to extrapolate with too few points.&#x22;);
            }
            return 2 * controlPoints[^1] - controlPoints[^2];
        }

        return controlPoints[i];
    }

    /// &#x3C;summary&#x3E;
    /// Calculates the rotation of a rider along the spline at time &#x3C;paramref name=&#x22;t&#x22;/&#x3E;.
    /// &#x3C;/summary&#x3E;
    /// &#x3C;param name=&#x22;t&#x22;&#x3E; The distance along the curve. &#x3C;/param&#x3E;
    /// &#x3C;returns&#x3E; The rotation of a rider along the curve. &#x3C;/returns&#x3E;
    /// &#x3C;exception cref=&#x22;IndexOutOfRangeException&#x22;&#x3E; Thrown when time &#x3C;paramref name=&#x22;t&#x22;/&#x3E; is out of range [0, 1]. &#x3C;/exception&#x3E;
    /// &#x3C;seealso cref=&#x22;SplineDisplacement&#x22;/&#x3E;
    /// &#x3C;seealso cref=&#x22;SplineVelocity&#x22;/&#x3E;
    /// &#x3C;seealso cref=&#x22;SplineAcceleration&#x22;/&#x3E;
    public Quaternion SplineRotation(float t)
    {
        return Quaternion.LookRotation(SplineVelocity(t));
    }

    /// &#x3C;summary&#x3E;
    /// Calculates the displacement along the spline at time &#x3C;paramref name=&#x22;t&#x22;/&#x3E;.
    /// &#x3C;/summary&#x3E;
    /// &#x3C;param name=&#x22;t&#x22;&#x3E; The distance along the curve. &#x3C;/param&#x3E;
    /// &#x3C;returns&#x3E; The displacement along the curve. &#x3C;/returns&#x3E;
    /// &#x3C;exception cref=&#x22;IndexOutOfRangeException&#x22;&#x3E; Thrown when time &#x3C;paramref name=&#x22;t&#x22;/&#x3E; is out of range [0, 1]. &#x3C;/exception&#x3E;
    /// &#x3C;seealso cref=&#x22;RiderPosition&#x22;/&#x3E;
    /// &#x3C;seealso cref=&#x22;SplineVelocity&#x22;/&#x3E;
    /// &#x3C;seealso cref=&#x22;SplineAcceleration&#x22;/&#x3E;
    public Vector3 SplineDisplacement(float t)
    {
        if (t &#x3C; 0 || t &#x3E; 1)
        {
            throw new IndexOutOfRangeException($&#x22;Time {t} out of bounds, should be in [0, 1].&#x22;);
        }
        if (t == 1)
        {
            return spline[^1].Evaluate(1.0f, 1.0f, 1.0f, 1.0f);
        }
        t *= spline.Count;
        return spline[(int)MathF.Floor(t)].Evaluate(1.0f, (t % 1.0f), (t % 1.0f) * (t % 1.0f), (t % 1.0f) * (t % 1.0f) * (t % 1.0f));
    }

    /// &#x3C;summary&#x3E;
    /// Calculates the velocity along the spline at time &#x3C;paramref name=&#x22;t&#x22;/&#x3E;.
    /// &#x3C;/summary&#x3E;
    /// &#x3C;param name=&#x22;t&#x22;&#x3E; The distance along the curve. &#x3C;/param&#x3E;
    /// &#x3C;returns&#x3E; The velocity along the curve. &#x3C;/returns&#x3E;
    /// &#x3C;exception cref=&#x22;IndexOutOfRangeException&#x22;&#x3E; Thrown when time &#x3C;paramref name=&#x22;t&#x22;/&#x3E; is out of range [0, 1]. &#x3C;/exception&#x3E;
    /// &#x3C;seealso cref=&#x22;SplineDisplacement&#x22;/&#x3E;
    /// &#x3C;seealso cref=&#x22;SplineAcceleration&#x22;/&#x3E;
    public Vector3 SplineVelocity(float t)
    {
        if (t &#x3C; 0 || t &#x3E; 1)
        {
            throw new IndexOutOfRangeException($&#x22;Time {t} out of bounds, should be in [0, 1].&#x22;);
        }
        if (t == 1)
        {
            return spline[^1].Evaluate(0.0f, 1.0f, 2.0f, 3.0f);
        }
        t *= spline.Count;
        return spline.Count * spline[(int)MathF.Floor(t)].Evaluate(0.0f, 1.0f, 2.0f * (t % 1.0f), 3.0f * (t % 1.0f) * (t % 1.0f));
    }

    /// &#x3C;summary&#x3E;
    /// Calculates the acceleration along the spline at time &#x3C;paramref name=&#x22;t&#x22;/&#x3E;.
    /// &#x3C;/summary&#x3E;
    /// &#x3C;param name=&#x22;t&#x22;&#x3E; The distance along the curve. &#x3C;/param&#x3E;
    /// &#x3C;returns&#x3E; The acceleration along the curve. &#x3C;/returns&#x3E;
    /// &#x3C;exception cref=&#x22;IndexOutOfRangeException&#x22;&#x3E; Thrown when time &#x3C;paramref name=&#x22;t&#x22;/&#x3E; is out of range [0, 1]. &#x3C;/exception&#x3E;
    /// &#x3C;seealso cref=&#x22;SplineDisplacement&#x22;/&#x3E;
    /// &#x3C;seealso cref=&#x22;SplineVelocity&#x22;/&#x3E;
    public Vector3 SplineAcceleration(float t)
    {
        if (t &#x3C; 0 || t &#x3E; 1)
        {
            throw new IndexOutOfRangeException($&#x22;Time {t} out of bounds, should be in [0, 1].&#x22;);
        }
        if (t == 1)
        {
            return spline[^1].Evaluate(0.0f, 0.0f, 2.0f, 6.0f);
        }
        t *= spline.Count;
        return spline.Count * spline.Count * spline[(int)MathF.Floor(t)].Evaluate(0.0f, 0.0f, 2.0f, 6.0f * (t % 1.0f));
    }

    /// &#x3C;summary&#x3E;
    /// Finds the t that approximately is closest to the provided position.
    /// &#x3C;/summary&#x3E;
    /// &#x3C;param name=&#x22;position&#x22;&#x3E; The position to measure from. &#x3C;/param&#x3E;
    /// &#x3C;returns&#x3E; The closest time. &#x3C;/returns&#x3E;
    public (float t, float direction) Mount(Vector3 position, Quaternion rotation)
    {
        float bestT = -1, bestDistance = float.PositiveInfinity;
        position = transform.InverseTransformPoint(position);
        foreach (Subline line in lines)
        {
            (float t, float distance) = line.ClosestGlobalT(position);

            if (distance &#x3C; bestDistance)
            {
                bestT = line.Evaluate(t).t;
                bestDistance = distance;
            }
        }

        return (bestT, Vector3.Angle(rotation * Vector3.forward, transform.TransformDirection(SplineVelocity(bestT))) &#x3C;= 90.0 ? 1 : -1);
    }

    /// &#x3C;summary&#x3E;
    /// Calculates the position of a rider along the spline at time &#x3C;paramref name=&#x22;t&#x22;/&#x3E;.
    /// &#x3C;/summary&#x3E;
    /// &#x3C;param name=&#x22;t&#x22;&#x3E; The distance along the curve in [0, 1] &#x3C;/param&#x3E;
    /// &#x3C;returns&#x3E; The position of a rider along the curve. &#x3C;/returns&#x3E;
    /// &#x3C;exception cref=&#x22;IndexOutOfRangeException&#x22;&#x3E; Thrown when time &#x3C;paramref name=&#x22;t&#x22;/&#x3E; is out of range [0, 1]. &#x3C;/exception&#x3E; 
    public Vector3 RiderPosition(float t) =&#x3E; transform.TransformPoint(SplineDisplacement(t) + SplineRotation(t) * Vector3.up * jointBounds.max.y);

    /// &#x3C;summary&#x3E;
    /// Calculates the rotation of a rider along the rail at time &#x3C;paramref name=&#x22;t&#x22;/&#x3E;.
    /// &#x3C;/summary&#x3E;
    /// &#x3C;param name=&#x22;t&#x22;&#x3E; The distance along the curve in [0, 1] &#x3C;/param&#x3E;
    /// &#x3C;returns&#x3E; The rotation of a rider along the curve. &#x3C;/returns&#x3E;
    /// &#x3C;exception cref=&#x22;IndexOutOfRangeException&#x22;&#x3E; Thrown when time &#x3C;paramref name=&#x22;t&#x22;/&#x3E; is out of range [0, 1]. &#x3C;/exception&#x3E; 
    public Quaternion RiderRotation(float t, float speed) =&#x3E; Quaternion.LookRotation(transform.TransformDirection(SplineVelocity(t) * speed));

    /// &#x3C;summary&#x3E;
    /// Calculates the speed of a rider along the rail at time &#x3C;paramref name=&#x22;t&#x22;/&#x3E;.
    /// &#x3C;/summary&#x3E;
    /// &#x3C;param name=&#x22;t&#x22;&#x3E; The distance along the curve in [0, 1] &#x3C;/param&#x3E;
    /// &#x3C;returns&#x3E; The speed of a rider along the curve. &#x3C;/returns&#x3E;
    /// &#x3C;exception cref=&#x22;IndexOutOfRangeException&#x22;&#x3E; Thrown when time &#x3C;paramref name=&#x22;t&#x22;/&#x3E; is out of range [0, 1]. &#x3C;/exception&#x3E; 
    public float RiderSpeed(float t) =&#x3E; SplineVelocity(t).magnitude;
}

/// &#x3C;summary&#x3E;
/// Implements rail editing via draggable control points.
/// &#x3C;/summary&#x3E;
[CustomEditor(typeof(SplineRail))]
public class SplineRailEditor : Editor
{
    void OnSceneGUI()
    {
        Handles.color = Color.blue;
        SplineRail rail = target as SplineRail;
        
        EditorGUI.BeginChangeCheck();
        for (int i = 0; i < rail.controlPoints.Length; i++)
        {
            rail.controlPoints[i] = rail.transform.InverseTransformPoint(Handles.PositionHandle(rail.transform.TransformPoint(rail.controlPoints[i]), Quaternion.identity));
        }
        if (EditorGUI.EndChangeCheck())
        {
            rail.CheckForUpdates();
        }
    }
}
    </code>
  </pre>
</details>
</div>
