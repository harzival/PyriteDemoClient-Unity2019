#PyriteDemoClient

Demo client application for [Pyrite3D](https://github.com/PyriteServer/PyriteServer/wiki). This project is also used to iterate on the initial Pyrite3D Unity client asset.

#License
Pyrite Demo Client source code is public under [MIT License](https://github.com/PyriteServer/PyriteDemoClient/blob/master/LICENSE).

- Updated to work with Unity 2018-2019 releases

# Main change to this repository and why


When using Unity 5.2 to run the Sample scene in PyriteDemoClient to
render the office's office mesh cubes, certain cubes don't get rendered.
When looking at debug console, error messages are shown to indicate a
failure loading the vertex data into a unity mesh, due to the verticies
ammounting to more than 65000 vertex for a specific cube.

Heres the code where the error is thrown, the line "m.vertices = Vertices;"
is where the error is thrown by Unity.Mesh's code.

GeometryBuffer.cs (code blocks starts at line 319) ...
---- 
	public void PopulateMeshes(GameObject gameObject, Material material)
        {
            if (!Processed) Process();

            if (Vertices.Length > 65000)
            {
                Debug.LogErrorFormat("GameObject {0} had too many vertices", gameObject.name);
            }

            var m = gameObject.GetComponent<MeshFilter>().mesh;
            m.vertices = Vertices;
            m.uv = UVs;
            m.triangles = Triangles;

            m.RecalculateNormals();

            if (material != null)
            {
                var renderer = gameObject.GetComponent<Renderer>();
                renderer.sharedMaterial = material;
            }
        }
----

Unity indexFormat documentation snippet (from 2019 docs)

"Index buffer can either be 16 bit (supports up to 65535 vertices in a mesh), 
or 32 bit (supports up to 4 billion vertices). Default index format is 16 
bit, since that takes less memory and bandwidth.

Note that GPU support for 32 bit indices is not guaranteed on all platforms; 
for example Android devices with Mali-400 GPU do not support them. 
When using 32 bit indices on such a platform, a warning message will be 
logged and mesh will not render."


The actual problem is Unity 5.2 doesn't have support for 32 bit mesh index
buffers, and the code for PyriteDemoClient was written before that support was
added.

We should take into consideration the extra memory overhead with using a 32 bit
mesh, and the fact our mesh's wont be anywhere near 65000 verticies if they
have been properly optimised to apply the surface detail with bump/normal maps.

For getting a proof of concept with the mesh data that is being used currently,
I have switched to a Unity 2019 stable release and changed the code to set
.indexFormat on a mesh to UnityEngine.Rendering.IndexFormat.UInt32; when it is
over 65000 vertices.

Code alterations:

GeometryBuffer.cs (code blocks starts at line 319) ...
----
        public void PopulateMeshes(GameObject gameObject, Material material)
        {
            if (!Processed) Process();

            var m = gameObject.GetComponent<MeshFilter>().mesh;
            if ( Vertices.Length > 65000 )
            {
                Debug.LogErrorFormat ( "Mesh {0} has over 65000 vertices, setting index buffer to Int32!", gameObject.name );
                m.indexFormat = UnityEngine.Rendering.IndexFormat.UInt32;
            }
            m.vertices = Vertices;
            m.uv = UVs;
            m.triangles = Triangles;

            m.RecalculateNormals();

            if (material != null)
            {
                var renderer = gameObject.GetComponent<Renderer>();
                renderer.sharedMaterial = material;
            }
        }
----

# Other changes
PyriteDemoClient/Assets/Editor/CrossPlatformInput/CrossPlatformInputInitialize.cs
- References to now-unsupported platforms to build to removed

PyriteDemoClient/Assets/Standard Assets/Cameras/Scripts/TargetFieldOfView.cs
- Changed line 62 from:
- if (!((r is TrailRenderer) || (r is ParticleRenderer) || (r is ParticleSystemRenderer)))
- to:
- if (!((r is TrailRenderer) || (r is ParticleSystemRenderer) || (r is ParticleSystemRenderer)))
- per changes to naming conventions for particle system classes
