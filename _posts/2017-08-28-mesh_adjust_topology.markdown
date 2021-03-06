---
layout: post
title:  "adjust mesh topology (remeshing) generated by Marching cube algorithms"
date:   2017-08-28 14:19:28
categories: graphics
---

Marching Cube is simple while results in right-angle triangles. 
I just implemented a very efficient remeshing method to remove such artifacts. The code depends on an external C++ header-only hash map library,
here is the link you can git-out [https://github.com/sparsehash]
```C++
/********************************************************************************************************/
	//initial topology
	/********************************************************************************************************/
	std::vector<double> vertex = mesh->vertex; // please use my matrix representation of mesh
	std::vector<int> face = mesh->face;
	int noTotalTriangles = mesh->noTotalTriangles;
	int noTotalVertex = mesh->noTotalVertex;
	sparse_hash_map< Vector2i, vector<int> > edgeTriangles;
	sparse_hash_map< Vector2i, vector<int> > edgeVertex;
	sparse_hash_map<int, vector<int>> vertexTriangles;
	sparse_hash_map<int, vector<Vector2i>> vertexEdges;
	for (size_t i = 0; i < noTotalTriangles; i++)
	{
		for (int j = 0; j < 3; j++)
		{
			int v1 = face[i * 3 + j % 3], v2 = face[i * 3 + (j + 1) % 3], v3 = face[i * 3 + (j + 2) % 3];
			Vector2i eKey = EdgeKey(v1, v2); // just a sorted 2d vector, do it as yours.
			vertexTriangles[v1].push_back(i);
			vertexTriangles[v2].push_back(i);
			vertexTriangles[v3].push_back(i);
			vertexEdges[v1].push_back(eKey);
			vertexEdges[v2].push_back(eKey);
			edgeTriangles[eKey].push_back(i);
			edgeVertex[eKey].push_back(v3);
		}
	}

	/********************************************************************************************************/
	//purturb vertex and edge flip considering angles
	/********************************************************************************************************/
	sparse_hash_map< Vector2i, vector<float> > edgeAngles;
	for (size_t i = 0; i < noTotalTriangles; i++)
	{
		Vector3f p1 = Vector3f(vertex[face[i * 3] * 3], vertex[face[i * 3] * 3 + 1], vertex[face[i * 3] * 3 + 2]);
		Vector3f p2 = Vector3f(vertex[face[i * 3 + 1] * 3], vertex[face[i * 3 + 1] * 3 + 1], vertex[face[i * 3 + 1] * 3 + 2]);
		Vector3f p3 = Vector3f(vertex[face[i * 3 + 2] * 3], vertex[face[i * 3 + 2] * 3 + 1], vertex[face[i * 3 + 2] * 3 + 2]);
		Vector3f angels = triangle_angles(p1, p2, p3);
		for (int j = 0; j < 3; j++)
		{
			int v1 = face[i * 3 + j % 3], v2 = face[i * 3 + (j + 1) % 3];
			Vector2i eKey = EdgeKey(v1, v2);
			edgeAngles[eKey].push_back(angels[(j + 2) % 3]);
		}
	}

	sparse_hash_map<int, int> processed_vertex;
	for (size_t i = 0; i < noTotalVertex; i++)
	{
		if (processed_vertex.contains(i)) continue;
		vector<Vector2i> half_edges = vertexEdges[i];
		sparse_hash_map<Vector2i, int> tmp_hfedg;
		Vector3f p(vertex[i * 3], vertex[i * 3 + 1], vertex[i * 3 + 2]);
		vector<int> nei_v;
		for (size_t j = 0; j < half_edges.size(); j++)
		{
			Vector2i eKey = half_edges[j];
			if (!tmp_hfedg.contains(eKey))
			{
				tmp_hfedg[eKey] = i;
				int neighbor_v = (eKey[0] == i) ? eKey[1] : eKey[0];
				nei_v.push_back(neighbor_v);
				if (edgeAngles[eKey].size() < 2) continue;
				if (fabs(edgeAngles[eKey][0] - 90.f) < 5.0f && fabs(edgeAngles[eKey][1] - 90.f) < 5.0f)
				{
					if (!processed_vertex.contains(neighbor_v))
					{
						int v3_0 = edgeVertex[eKey][0];
						Vector3f p3_0(vertex[v3_0 * 3], vertex[v3_0 * 3 + 1], vertex[v3_0 * 3 + 2]);
						float radius = length(p3_0 - p);
						Vector3f neighbor_p(vertex[neighbor_v * 3], vertex[neighbor_v * 3 + 1], vertex[neighbor_v * 3 + 2]);
						neighbor_p = p + (neighbor_p - p).normalised() * radius;
						vertex[neighbor_v * 3] = neighbor_p[0]; vertex[neighbor_v * 3 + 1] = neighbor_p[1];
						vertex[neighbor_v * 3 + 2] = neighbor_p[2];
					}
					else
					{
						vector<int> tri = edgeTriangles[eKey];
						int v3_0 = edgeVertex[eKey][0];
						int v3_1 = edgeVertex[eKey][1];
						vector<int> indices;
						for (size_t k = 0; k < 3; k++)indices.push_back(face[tri[0] * 3 + k]);
						vector<int>::iterator itt = std::find(indices.begin(), indices.end(), v3_0);
						int pos_1 = std::distance(indices.begin(), itt); // watch out, this is slow
						itt = std::find(indices.begin(), indices.end(), neighbor_v);
						int pos_2 = std::distance(indices.begin(), itt);
						int pos_3 = 0;
						if (pos_1 + pos_2 == 1) pos_3 = 2;
						else if (pos_1 + pos_2 == 2) pos_3 = 1;
						else if (pos_1 + pos_2 == 3) pos_3 = 0;
						face[tri[0] * 3 + pos_1] = v3_0; face[tri[0] * 3 + pos_2] = neighbor_v;
						face[tri[0] * 3 + pos_3] = v3_1;

						indices.clear();
						for (size_t k = 0; k < 3; k++)indices.push_back(face[tri[1] * 3 + k]);
						itt = std::find(indices.begin(), indices.end(), v3_1);
					    pos_1 = std::distance(indices.begin(), itt);
						itt = std::find(indices.begin(), indices.end(), i);
						pos_2 = std::distance(indices.begin(), itt);
						if (pos_1 + pos_2 == 1) pos_3 = 2;
						else if (pos_1 + pos_2 == 2) pos_3 = 1;
						else if (pos_1 + pos_2 == 3) pos_3 = 0;
						face[tri[1] * 3 + pos_1] = v3_1; face[tri[1] * 3 + pos_2] = i;
						face[tri[1] * 3 + pos_3] = v3_0;
					}
				}
			}
		}
		for (size_t k = 0; k < nei_v.size(); k++)
		{
			processed_vertex[nei_v[k]] = 1;
		}
	}
	/********************************************************************************************************/
	//renew topology
	/********************************************************************************************************/
	mesh->vertex = vertex;
	mesh->face = face;
	mesh->noTotalTriangles = face.size() / 3;
	mesh->noTotalVertex = vertex.size() / 3;
	noTotalTriangles = mesh->noTotalTriangles;
	noTotalVertex = mesh->noTotalVertex;
	edgeVertex.clear();
	vertexTriangles.clear();
	vertexEdges.clear();
	edgeTriangles.clear();
	for (size_t i = 0; i < noTotalTriangles; i++)
	{
		for (int j = 0; j < 3; j++)
		{
			int v1 = face[i * 3 + j % 3], v2 = face[i * 3 + (j + 1) % 3], v3 = face[i * 3 + (j + 2) % 3];
			Vector2i eKey = EdgeKey(v1, v2);
			vertexEdges[v1].push_back(eKey);
			vertexEdges[v2].push_back(eKey);
			edgeVertex[eKey].push_back(v3);
		}
	}
	/********************************************************************************************************/
	//adjust vertex considering 1-ring edge lengths
	/********************************************************************************************************/
	for (size_t iter = 0; iter < 3; iter++)
	{
		for (size_t i = 0; i < noTotalVertex; i++)
		{
			vector<Vector2i> half_edges = vertexEdges[i];
			sparse_hash_map<Vector2i, int> tmp_hfedg;
			float sum_weight = 0.f;
			Vector3f sum_diff(0.f, 0.f, 0.f);
			Vector3f p(vertex[i * 3], vertex[i * 3 + 1], vertex[i * 3 + 2]);
			for (size_t j = 0; j < half_edges.size(); j++)
			{
				Vector2i eKey = half_edges[j];
				if (!tmp_hfedg.contains(eKey))
				{
					tmp_hfedg[eKey] = i;
					float w = 0.f;
					int neighbor_v = (eKey[0] == i) ? eKey[1] : eKey[0];
					Vector3f neighbor_p(vertex[neighbor_v * 3], vertex[neighbor_v * 3 + 1], vertex[neighbor_v * 3 + 2]);
#ifdef UMBRELLA
					w = 1.f;
#endif

#ifdef SPRING
					float w_0 = 0.0f;
					float w_1 = 0.0f;
					vector<int> verts = edgeVertex[eKey];
					Vector3f p3_0(vertex[verts[0] * 3], vertex[verts[0] * 3 + 1], vertex[verts[0] * 3 + 2]);
					float area_0 = length(cross(p3_0 - p, p3_0 - neighbor_p)); // you have to do your vector operations
					if (area_0 == 0.f) w_0 = 0.f;
					else
					{
						w_0 = dot(p3_0 - p, p3_0 - p) + dot(p3_0 - neighbor_p, p3_0 - neighbor_p) - dot(neighbor_p - p, neighbor_p - p);
						w_0 /= 0.5 * area_0;
					}
					//w_0 = dot((p - p3_0), (neighbor_p - p3_0)) / length(cross((p - p3_0), (neighbor_p - p3_0)));
					w = w_0;
					if (verts.size() == 2)
					{
						Vector3f p3_1(vertex[verts[1] * 3], vertex[verts[1] * 3 + 1], vertex[verts[1] * 3 + 2]);
						float area_1 = length(cross(p3_1 - p, p3_1 - neighbor_p));
						if (area_1 == 0.f) w_1 = 0.f;
						else
						{
							w_1 = dot(p3_1 - p, p3_1 - p) + dot(p3_1 - neighbor_p, p3_1 - neighbor_p) - dot(neighbor_p - p, neighbor_p - p);
							w_1 /= 0.5 * area_1;
							//w_1 = dot((p - p3_1), (neighbor_p - p3_1)) / length(cross((p - p3_1), (neighbor_p - p3_1)));
						}
						w += w_1;
						w *= 0.5f;
					}
#endif
					sum_diff += (neighbor_p - p) * w;
					sum_weight += w;
				}
			}
#ifdef SPRING
			if (sum_weight != 0.f)
#endif
			{
				Vector3f avg_pos = p + sum_diff / sum_weight;
				vertex[i * 3] = avg_pos[0]; vertex[i * 3 + 1] = avg_pos[1]; vertex[i * 3 + 2] = avg_pos[2];
			}
		}
	}
  ```

[https://github.com/sparsehash]: https://github.com/sparsehash
