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
  <h2 id="splinerail" align=middle><i>Sandlocked</i></h2>
</p>

<pre>
<code>
using System;
using System.Collections.Generic;
using System.Linq;
using Unity.Mathematics;
using UnityEditor;
using UnityEngine;

/// <summary>
/// Represents a grindable rail along a Catmull-Rom spline.
/// </summary>
[ExecuteInEditMode, RequireComponent(typeof(MeshFilter)), RequireComponent(typeof(MeshCollider))]
public class SplineRail : MonoBehaviour, IRail
{
    struct Subcurve
    {
        Vector3[] factors;

        /// <summary>
        /// Constructs a subcurve from 4 control points;
        /// </summary>
        /// <param name="controlPoint1"> First control point. </param>
        /// <param name="controlPoint2"> Second control point. </param>
        /// <param name="controlPoint3"> Third control point. </param>
        /// <param name="controlPoint4"> Fourth control point. </param>
        public Subcurve(Vector3 controlPoint1, Vector3 controlPoint2, Vector3 controlPoint3, Vector3 controlPoint4)
        {
            factors = new Vector3[4];
            factors[0] = controlPoint2;
            factors[1] = (controlPoint3 - controlPoint1) / 2.0f;
            factors[2] = controlPoint1 + 2.0f * controlPoint3 - (5.0f * controlPoint2 + controlPoint4) / 2.0f;
            factors[3] = (3 * controlPoint2 + controlPoint4 - controlPoint1 - 3.0f * controlPoint3) / 2.0f;
        }

        /// <summary>
        /// Evaluates the curve at a collection of t values.
        /// </summary>
        /// <param name="t1"> The first t value. </param>
        /// <param name="t2"> The second t value. </param>
        /// <param name="t3"> The third t value.</param>
        /// <param name="t4"> The fourth t value.</param>
        /// <returns> The value along the curve. </returns>
        public readonly Vector3 Evaluate(float t1, float t2, float t3, float t4)
        {
            return t1 * factors[0] + t2 * factors[1] + t3 * factors[2] + t4 * factors[3];
        }
    }

    struct Subline
    {
        Vector3 start, path;
        float startT, pathT;

        /// <summary>
        /// Constructs a subline from a start and a end.
        /// </summary>
        /// <param name="start"> The start of the curve. </param>
        /// <param name="end"> The end of the curve. </param>
        public Subline(Vector3 start, float startT, Vector3 end, float endT)
        {
            this.start = start;
            path = end - start;
            this.startT = startT;
            pathT = endT - startT;
        }

        /// <summary>
        /// Constructs a subline from a start and a end.
        /// </summary>
        /// <param name="start"> The start of the curve. </param>
        /// <param name="end"> The end of the curve. </param>
        public Subline((Vector3 point, float t) start, (Vector3 point, float t) end)
        {
            (this.start, startT) = start;
            path = end.point - start.point;
            pathT = end.t - start.t;
        }

        /// <summary>
        /// Evaluates the line at a t value.
        /// </summary>
        /// <param name="t"> The t value. </param>
        /// <returns> The value along the curve. </returns>
        public readonly (Vector3 point, float t) Evaluate(float t)
        {
            return (start + t * path, startT + t * pathT);
        }

        /// <summary>
        /// Find the closest global t to a position.
        /// </summary>
        /// <param name="position"> the position to the closest t to. </param>
        /// <returns> The tuple of the closest global t and the distance to that t. </returns>
        public readonly (float, float) ClosestGlobalT(Vector3 position)
        {
            (Vector3 point, float t) = Evaluate(Mathf.Clamp(Vector3.Dot(position - start, path) / path.sqrMagnitude, 0.0f, 1.0f));
            return (t, (position - point).magnitude);
        }
    }

    /// <summary>
    /// The size of the step between joints.
    /// </summary>
    [Header("Configuration")]
    public float stepLength = 0.5f;
    int stepSizeHash;

    /// <summary>
    /// How many substeps to take between joints.
    /// </summary>
    public int precision = 10;
    int precisionHash;

    /// <summary>
    /// The Catmull-Rom spline as control points.
    /// </summary>
    public Vector3[] controlPoints = new Vector3[]
    {
        new (-1.0f, 0.0f),
        new (1.0f, 0.0f)
    };
    int controlPointsHash;

    /// <summary>
    /// Reference joint along the rail.
    /// </summary>
    public Vector2[] joint = new Vector2[]
    {
        new (0.3f, 0.3f),
        new (-0.3f, 0.3f),
        new (-0.3f, -0.3f),
        new (0.3f, -0.3f),
    };
    int jointHash;

    Bounds jointBounds;


    readonly List<Subcurve> spline = new();
    readonly List<Subline> lines = new();

    void Update()
    {
        int controlPointsHash = controlPoints.GetHashCode(),
        stepSizeHash = stepLength.GetHashCode(),
        precisionHash = precision.GetHashCode(),
        jointHash = joint.GetHashCode();

        if (this.controlPointsHash != controlPointsHash || this.stepSizeHash != stepSizeHash || this.precisionHash != precisionHash)
        {
            PreprocessSpline();
            controlPointsHash = controlPoints.GetHashCode();
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
            PreprocessJoint();
            this.jointHash = jointHash;
        }
    }

    void PreprocessSpline()
    {
        if (precision == 0 || stepLength == 0)
        {
            throw new ArgumentException("Precision and stepLength must not be 0.");
        }

        spline.Clear();

        for (int i = 0; i + 1 < controlPoints.Length; i++)
        {
            spline.Add(new(GetControlPoint(i - 1), GetControlPoint(i), GetControlPoint(i + 1), GetControlPoint(i + 2)));
        }

        lines.Clear();

        float t = 0.0f;

        for (int i = 0; i < precision; i++)
        {
            t += stepLength / (precision * SplineVelocity(t).magnitude);
            t = Mathf.Clamp(t, 0.0f, 1.0f);
        }

        lines.Add(new(SplineDisplacement(0.0f), 0.0f, SplineDisplacement(t), t));

        while (t < 1.0f)
        {
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
        jointBounds = new();
        foreach (Vector3 point in joint)
        {
            jointBounds.Encapsulate(point);
        }
    }

    /// <summary>
    /// Updates <c> mesh </c> to reflect <c> controlPoints </c>.
    /// </summary>
    public void UpdateMesh()
    {
        PreprocessSpline();

        if (spline.Count == 0)
        {
            return;
        }

        List<Vector3> vertices = new();
        List<int> triangles = new();

        Quaternion direction = SplineRotation(0.0f);
        Vector3 displacement = GetControlPoint(0);
        for (int i = 0; i < joint.Count(); i++)
        {
            vertices.Add(direction * (Vector3)joint[i] + displacement);
        }

        vertices.Add(direction * (Vector3)joint[0] + displacement);
        for (int i = 1; i < joint.Count(); i++)
        {
            vertices.Add(direction * (Vector3)joint[i] + displacement);
            vertices.Add(direction * (Vector3)joint[i] + displacement);
        }
        vertices.Add(direction * (Vector3)joint[0] + displacement);

        foreach (Subline line in lines)
        {
            for (int i = vertices.Count; i < vertices.Count + 2 * joint.Count(); i += 2)
            {
                triangles.Add(i - 8);
                triangles.Add(i - 7);
                triangles.Add(i + 1);

                triangles.Add(i + 1);
                triangles.Add(i);
                triangles.Add(i - 8);
            }

            float t = line.Evaluate(1.0f).t;

            direction = SplineRotation(t);
            displacement = SplineDisplacement(t);
            vertices.Add(direction * (Vector3)joint[0] + displacement);
            for (int i = 1; i < joint.Count(); i++)
            {
                vertices.Add(direction * (Vector3)joint[i] + displacement);
                vertices.Add(direction * (Vector3)joint[i] + displacement);
            }
            vertices.Add(direction * (Vector3)joint[0] + displacement);
        }

        direction = SplineRotation(1.0f);
        displacement = controlPoints[^1];
        for (int i = 0; i < joint.Count(); i++)
        {
            vertices.Add(direction * (Vector3)joint[i] + displacement);
        }

        Mesh mesh = new() { vertices = vertices.ToArray(), triangles = triangles.ToArray() };
        mesh.Optimize();
        mesh.RecalculateNormals();
        TryGetComponent(out MeshFilter filter);
        filter.mesh = mesh;
        TryGetComponent(out MeshCollider collider);
        collider.sharedMesh = mesh;
    }

    /// <summary>
    /// Retrieves the <paramref name="i"/>th control point along the spline. Will extrapolate one point ahead or behind.
    /// </summary>
    /// <param name="i"> Index of the control point to retrieve. </param>
    /// <returns> <paramref name="i"/>th control point. </returns>
    /// <exception cref="IndexOutOfRangeException"> Thrown when index <paramref name="i"/> is out of range [-1, <c>controlPoints.Count</c>]. </exception>
    /// <exception cref="IndexOutOfRangeException"> Thrown when there are too few points to extrapolate (less than 2) and the index is on the bounds. </exception>
    public Vector3 GetControlPoint(int i)
    {
        if (i < -1 || i > controlPoints.Length)
        {
            throw new IndexOutOfRangeException($"Index {i} out of bounds, should be in [-1 and {controlPoints.Length}].");
        }

        if (i == -1)
        {
            if (controlPoints.Length < 2)
            {
                throw new IndexOutOfRangeException($"Tried to extrapolate with too few points.");
            }
            return 2 * controlPoints[0] - controlPoints[1];
        }


        if (i == controlPoints.Length)
        {
            if (controlPoints.Length < 2)
            {
                throw new ArgumentException($"Tried to extrapolate with too few points.");
            }
            return 2 * controlPoints[^1] - controlPoints[^2];
        }

        return controlPoints[i];
    }

    /// <summary>
    /// Calculates the rotation of a rider along the spline at time <paramref name="t"/>.
    /// </summary>
    /// <param name="t"> The distance along the curve. </param>
    /// <returns> The rotation of a rider along the curve. </returns>
    /// <exception cref="IndexOutOfRangeException"> Thrown when time <paramref name="t"/> is out of range [0, 1]. </exception>
    /// <seealso cref="SplineDisplacement"/>
    /// <seealso cref="SplineVelocity"/>
    /// <seealso cref="SplineAcceleration"/>
    public Quaternion SplineRotation(float t)
    {
        return Quaternion.LookRotation(SplineVelocity(t));
    }

    /// <summary>
    /// Calculates the displacement along the spline at time <paramref name="t"/>.
    /// </summary>
    /// <param name="t"> The distance along the curve. </param>
    /// <returns> The displacement along the curve. </returns>
    /// <exception cref="IndexOutOfRangeException"> Thrown when time <paramref name="t"/> is out of range [0, 1]. </exception>
    /// <seealso cref="RiderPosition"/>
    /// <seealso cref="SplineVelocity"/>
    /// <seealso cref="SplineAcceleration"/>
    public Vector3 SplineDisplacement(float t)
    {
        if (t < 0 || t > 1)
        {
            throw new IndexOutOfRangeException($"Time {t} out of bounds, should be in [0, 1].");
        }
        if (t == 1)
        {
            return spline[^1].Evaluate(1.0f, 1.0f, 1.0f, 1.0f);
        }
        t *= spline.Count;
        return spline[(int)MathF.Floor(t)].Evaluate(1.0f, (t % 1.0f), (t % 1.0f) * (t % 1.0f), (t % 1.0f) * (t % 1.0f) * (t % 1.0f));
    }

    /// <summary>
    /// Calculates the velocity along the spline at time <paramref name="t"/>.
    /// </summary>
    /// <param name="t"> The distance along the curve. </param>
    /// <returns> The velocity along the curve. </returns>
    /// <exception cref="IndexOutOfRangeException"> Thrown when time <paramref name="t"/> is out of range [0, 1]. </exception>
    /// <seealso cref="SplineDisplacement"/>
    /// <seealso cref="SplineAcceleration"/>
    public Vector3 SplineVelocity(float t)
    {
        if (t < 0 || t > 1)
        {
            throw new IndexOutOfRangeException($"Time {t} out of bounds, should be in [0, 1].");
        }
        if (t == 1)
        {
            return spline[^1].Evaluate(0.0f, 1.0f, 2.0f, 3.0f);
        }
        t *= spline.Count;
        return spline.Count * spline[(int)MathF.Floor(t)].Evaluate(0.0f, 1.0f, 2.0f * (t % 1.0f), 3.0f * (t % 1.0f) * (t % 1.0f));
    }

    /// <summary>
    /// Calculates the acceleration along the spline at time <paramref name="t"/>.
    /// </summary>
    /// <param name="t"> The distance along the curve. </param>
    /// <returns> The acceleration along the curve. </returns>
    /// <exception cref="IndexOutOfRangeException"> Thrown when time <paramref name="t"/> is out of range [0, 1]. </exception>
    /// <seealso cref="SplineDisplacement"/>
    /// <seealso cref="SplineVelocity"/>
    public Vector3 SplineAcceleration(float t)
    {
        if (t < 0 || t > 1)
        {
            throw new IndexOutOfRangeException($"Time {t} out of bounds, should be in [0, 1].");
        }
        if (t == 1)
        {
            return spline[^1].Evaluate(0.0f, 0.0f, 2.0f, 6.0f);
        }
        t *= spline.Count;
        return spline.Count * spline.Count * spline[(int)MathF.Floor(t)].Evaluate(0.0f, 0.0f, 2.0f, 6.0f * (t % 1.0f));
    }

    /// <summary>
    /// Finds the t that approximately is closest to the provided position.
    /// </summary>
    /// <param name="position"> The position to measure from. </param>
    /// <returns> The closest time. </returns>
    public (float t, float direction) Mount(Vector3 position, Quaternion rotation)
    {
        float bestT = -1, bestDistance = float.PositiveInfinity;
        position = transform.InverseTransformPoint(position);
        foreach (Subline line in lines)
        {
            (float t, float distance) = line.ClosestGlobalT(position);

            if (distance < bestDistance)
            {
                bestT = line.Evaluate(t).t;
                bestDistance = distance;
            }
        }

        return (bestT, Vector3.Angle(rotation * Vector3.forward, transform.TransformDirection(SplineVelocity(bestT))) <= 90.0 ? 1 : -1);
    }

    /// <summary>
    /// Calculates the position of a rider along the spline at time <paramref name="t"/>.
    /// </summary>
    /// <param name="t"> The distance along the curve in [0, 1] </param>
    /// <returns> The position of a rider along the curve. </returns>
    /// <exception cref="IndexOutOfRangeException"> Thrown when time <paramref name="t"/> is out of range [0, 1]. </exception> 
    public Vector3 RiderPosition(float t) => transform.TransformPoint(SplineDisplacement(t) + SplineRotation(t) * Vector3.up * jointBounds.max.y);

    /// <summary>
    /// Calculates the rotation of a rider along the rail at time <paramref name="t"/>.
    /// </summary>
    /// <param name="t"> The distance along the curve in [0, 1] </param>
    /// <returns> The rotation of a rider along the curve. </returns>
    /// <exception cref="IndexOutOfRangeException"> Thrown when time <paramref name="t"/> is out of range [0, 1]. </exception> 
    public Quaternion RiderRotation(float t, float speed) => Quaternion.LookRotation(transform.TransformDirection(SplineVelocity(t) * speed));

    /// <summary>
    /// Calculates the speed of a rider along the rail at time <paramref name="t"/>.
    /// </summary>
    /// <param name="t"> The distance along the curve in [0, 1] </param>
    /// <returns> The speed of a rider along the curve. </returns>
    /// <exception cref="IndexOutOfRangeException"> Thrown when time <paramref name="t"/> is out of range [0, 1]. </exception> 
    public float RiderSpeed(float t) => SplineVelocity(t).magnitude;
}

/// <summary>
/// Implements rail editing via draggable control points.
/// </summary>
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
            rail.UpdateMesh();
        }
    }
}
</code>
</pre>
</div>
