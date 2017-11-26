---
layout: post
title:  "uniform and adaptive Loop-color-displacement subdivision"
date:   2017-08-28 15:21:28
categories: graphics
---

Share a simple program that could be used to recover surface details from a color image, very useful conque intrisinc limitations of 3d reconstrcution on structure light sensors

```C++

#pragma once

#include "color_gradient_domain.h"
#include "MatrixMesh.h"


void genDismap();
int attenuation(double w, int subdiv);
void adaptive_lzp_details(
	std::vector<double> & vertices_in, std::vector<double> & texCoords_in,
	std::vector<double> & normals_in, std::vector<int> & face_in);

void genDismap()
{
//to do as your wish
}

int attenuation(double w, int subdiv)
{
	double x = w * std::pow(2, -subdiv);
	int level = x > 1. ? 1 : 0;
	return level;
}

void adaptive_lzp_details(
	std::vector<double> & vertices_in, std::vector<double>& texCoords_in, 
	std::vector<double> & normals_in, std::vector<int> & face_in)
{   
	cv::Mat smooth_gray_invert;
	genDismap(color_image, smooth_gray_invert); // for example, invert gray as final displacement map
	int tex_w = smooth_gray_invert.cols;
	int tex_h = smooth_gray_invert.rows;
	std::vector<Vector3f> vertices;
	std::vector<Vector2f> texcors;
	for (size_t i = 0; i < vertices_in.size() / 3; i++)
	{
		vertices.push_back(Vector3f(vertices_in[i * 3], vertices_in[i * 3 + 1], vertices_in[i * 3 + 2]));
	}
	for (size_t i = 0; i < texCoords_in.size() / 2; i++)
	{
		texcors.push_back(Vector2f(texCoords_in[i * 2], texCoords_in[i * 2 + 1]));
	}
	int noTotalVertex = 0;
	int noTotalTriangles = 0;
	/********************************************************/
	//adaptive and uniform Loop subdivision //
	/*******************************************************/
	float s_weight = 0.1f;
	double threshold = 10.;
	double min_area = 0.01;
	int max_subdiv = 3;
	for (size_t n_subdiv = 0; n_subdiv < max_subdiv; ++n_subdiv)
	{
		sparse_hash_map< Vector2i, vector<int> > edgeTriangles;
		sparse_hash_map< Vector2i, vector<int> > edgeVertex;
		sparse_hash_map<int, vector<Vector2i>> vertexEdges;
		noTotalTriangles = face_in.size() / 3;
		for (size_t i = 0; i < noTotalTriangles; i++)
		{
			for (int j = 0; j < 3; j++)
			{
				int v1 = face_in[i * 3 + j % 3], v2 = face_in[i * 3 + (j + 1) % 3], v3 = face_in[i * 3 + (j + 2) % 3];
				Vector2i eKey(0, 0);
				if (v1 < v2)  eKey = Vector2i(v1, v2);
				else  eKey = Vector2i(v2, v1);
				vertexEdges[v1].push_back(eKey);
				vertexEdges[v2].push_back(eKey);
				edgeTriangles[eKey].push_back(i);
				edgeVertex[eKey].push_back(v3);
			}
		}
		/********************************************************************************************************/
		//triangle-triangles structure
		/********************************************************************************************************/
		sparse_hash_map<Vector2i, int> borderEdges;
		sparse_hash_map< int, vector<int> > triTriangles;
		for (sparse_hash_map< Vector2i, vector<int> >::iterator it = edgeTriangles.begin(); it != edgeTriangles.end(); ++it)
		{
			vector<int> tris = it->second;
			if (tris.size() == 1) borderEdges[it->first] = 1;
			else
			{
				triTriangles[tris[0]].push_back(tris[1]); triTriangles[tris[1]].push_back(tris[0]);
			}
		}
		/********************************************************************************************************/
		//check if subdivision
		/********************************************************************************************************/
		sparse_hash_map<Vector2i, int> splitEdges;
		vector<Vector2i> tmp_edges;
		sparse_hash_map<int, int> processed_tri;
		for (size_t i = 0; i < noTotalTriangles; i++)
		{
#ifdef ADAPTIVE
			double g_weight = 0.0;
			int v1 = face_in[i * 3];
			Vector3f p1 = vertices[v1];
			int v2 = face_in[i * 3 + 1];
			Vector3f p2 = vertices[v2];
			int v3 = face_in[i * 3 + 2];
			Vector3f p3 = vertices[v3];
			Vector3f n = cross(p2 - p1, p3 - p1);
			double area = std::sqrt(n[0] * n[0] + n[1] * n[1] + n[2] * n[2]) / 2.;
			if (area < min_area) continue;
			g_weight += getSubPixel(smooth_gray_invert, texcors[v1][0], texcors[v1][1], tex_w, tex_h);
			g_weight += getSubPixel(smooth_gray_invert, texcors[v2][0], texcors[v2][1], tex_w, tex_h);
			g_weight += getSubPixel(smooth_gray_invert, texcors[v3][0], texcors[v3][1], tex_w, tex_h);
			int level = attenuation(g_weight, n_subdiv);
			if (level < 1) continue;
			Vector3f n1 = n.normalised();
			vector<int> nei_tris = triTriangles[i];
			for (size_t k = 0; k < nei_tris.size(); k++)
			{
				int id = nei_tris[k];
				v1 = face_in[id * 3];
				p1 = vertices[v1];
				v2 = face_in[id * 3 + 1];
				p2 = vertices[v2];
				v3 = face_in[id * 3 + 2];
				p3 = vertices[v3];
				Vector3f n2 = cross(p2 - p1, p3 - p1).normalised();
				double r = n1[0] * n2[0] + n1[1] * n2[1] + n1[2] * n2[2];
				if (r > 1.0) {
					r = 1.0;
				}
				if (r < -1.0) {
					r = -1.0;
				}
				if (std::acos(r) / M_PI*180.0 > threshold)
				{
					if (!processed_tri.contains(id)) for (int j = 0; j < 3; j++)
					{
						int v1 = face_in[id * 3 + j % 3], v2 = face_in[id * 3 + (j + 1) % 3], v3 = face_in[id * 3 + (j + 2) % 3];
						Vector2i eKey(0, 0);
						if (v1 < v2)  eKey = Vector2i(v1, v2);
						else  eKey = Vector2i(v2, v1);
						if (!splitEdges.contains(eKey))
						{
							splitEdges[eKey] = 1;
							tmp_edges.push_back(eKey);
						}
						processed_tri[id] = 1;
					}
				}
			}
#endif
#ifdef ADAPTIVE
			if (!processed_tri.contains(i))
			{
#endif
				for (int j = 0; j < 3; j++)
				{
					int v1 = face_in[i * 3 + j % 3], v2 = face_in[i * 3 + (j + 1) % 3], v3 = face_in[i * 3 + (j + 2) % 3];
					Vector2i eKey(0, 0);
					if (v1 < v2)  eKey = Vector2i(v1, v2);
					else  eKey = Vector2i(v2, v1);
					if (!splitEdges.contains(eKey))
					{
						splitEdges[eKey] = 1;
						tmp_edges.push_back(eKey);
					}
					processed_tri[i] = 1;
				}
#ifdef ADAPTIVE
			}
#endif
		}
#ifdef ADAPTIVE
		processed_tri.clear();
#endif
		/********************************************************************************************************/
		//do subdivision
		/********************************************************************************************************/
		noTotalVertex = vertices.size();
		// compute edge vertex
		for (size_t i = 0; i< tmp_edges.size(); i++)
		{
			Vector2i eKey = tmp_edges[i];
			Vector3f p0 = vertices[eKey[0]];
			Vector3f p1 = vertices[eKey[1]];
			Vector2f uv0 = texcors[eKey[0]];
			Vector2f uv1 = texcors[eKey[1]];
			Vector3f edge_p(0.f, 0.f, 0.f);
			Vector2f edge_uv(0.f, 0.f);
			if (borderEdges.contains(eKey))
			{
				edge_p = (p0 + p1) * 0.5f;
				vertices.push_back(edge_p);
				edge_uv = (uv0 + uv1) * 0.5;
				texcors.push_back(edge_uv);
			}
			else
			{
				vector<int> verts = edgeVertex[eKey];
				Vector3f p3_0 = vertices[verts[0]];
				Vector3f p3_1 = vertices[verts[1]];
				edge_p = (p0 + p1) * (3.0f / 8.0f) + (p3_0 + p3_1) * (1.0f / 8.0f);
				vertices.push_back(edge_p);
				Vector2f uv3_0 = texcors[verts[0]];
				Vector2f uv3_1 = texcors[verts[1]];
				edge_uv = (uv0 + uv1) * (3.0f / 8.0f) + (uv3_0 + uv3_1) * (1.0f / 8.0f);
				texcors.push_back(edge_uv);
			}
			splitEdges[eKey] = i + noTotalVertex;
		}
		tmp_edges.clear();
		//adjust source vertex
		for (size_t i = 0; i < noTotalVertex; i++)
		{
			vector<Vector2i> half_edges = vertexEdges[i];
			sparse_hash_map<Vector2i, int> tmp_hfedg;
			vector<int> neighbor_vs;
			Vector3f p = vertices[i];
			Vector2f uv_p = texcors[i];
			bool on_border = false;
			vector<int> border_vertex;
			for (size_t j = 0; j < half_edges.size(); j++)
			{
				Vector2i eKey = half_edges[j];
				if (!tmp_hfedg.contains(eKey))
				{
					tmp_hfedg[eKey] = i;
					int neighbor_v = (eKey[0] == i) ? eKey[1] : eKey[0];
					neighbor_vs.push_back(neighbor_v);
					if (borderEdges.contains(eKey)) {
						on_border = true; border_vertex.push_back(neighbor_v);
					}
				}
			}
			Vector3f p_new(0.f, 0.f, 0.f);
			Vector2f uv_new(0.f, 0.f);
			if (on_border)
			{
				p_new = vertices[i] * 0.75 + (vertices[border_vertex[0]] + vertices[border_vertex[1]]) * 0.125;
				uv_new = texcors[i] * 0.75 + (texcors[border_vertex[0]] + texcors[border_vertex[1]]) * 0.125;
			}
			else
			{
				int n_ring = neighbor_vs.size();
				float beta = 0.f;
				if (n_ring > 3)
				{
					beta = 3.0f / (8.0f * n_ring);
				}
				else if (n_ring == 3)
				{
					beta = 3.0f / 16.0f;
				}
				for (size_t k = 0; k < n_ring; k++)
				{
					Vector3f neighbor_p = vertices[neighbor_vs[k]];
					Vector2f neighbor_uv = texcors[neighbor_vs[k]];
					p_new += neighbor_p * beta;
					uv_new += neighbor_uv * beta;
				}
				p_new += p * (1.0f - n_ring * beta);
				uv_new += uv_p * (1.0f - n_ring * beta);
			}
			vertices[i] = p_new;
			texcors[i] = uv_new;
		}

		// split as new faces
		for (size_t i = 0; i < noTotalTriangles; i++)
		{
			int num_splits = 0;
			vector<int> has_edge_vertex(3, -1);
			int pinV = 0;
			vector<int> evs;
			vector<int> indices;
			for (size_t j = 0; j < 3; j++)
			{
				indices.push_back(face_in[i * 3 + j]);
				int v1 = face_in[i * 3 + j % 3], v2 = face_in[i * 3 + (j + 1) % 3], v3 = face_in[i * 3 + (j + 2) % 3];
				Vector2i eKey(0, 0);
				if (v1 < v2)  eKey = Vector2i(v1, v2);
				else  eKey = Vector2i(v2, v1);
				if (splitEdges.contains(eKey))
				{
					has_edge_vertex[j] = 1;
					pinV = v3;
					evs.push_back(splitEdges[eKey]);
					++num_splits;
				}
			}
			if (num_splits == 1)
			{
				vector<int>::iterator itt = std::find(indices.begin(), indices.end(), pinV);
				int pos_3 = std::distance(indices.begin(), itt);
				int v_1 = indices[(pos_3 + 1) % 3];
				int v_2 = indices[(pos_3 + 2) % 3];
				face_in[i * 3] = pinV; face_in[i * 3 + 1] = v_1; face_in[i * 3 + 2] = evs[0];
				face_in.push_back(evs[0]); face_in.push_back(v_2); face_in.push_back(pinV);
			}
			else if (num_splits == 2)
			{
				vector<int>::iterator itt = std::find(indices.begin(), indices.end(), pinV);
				int pos_3 = std::distance(indices.begin(), itt);
				int v_1 = indices[(pos_3 + 1) % 3];
				int v_2 = indices[(pos_3 + 2) % 3];
				if (has_edge_vertex[0] == 1 && has_edge_vertex[2] == 1) {
					face_in[i * 3] = pinV; face_in[i * 3 + 1] = v_1; face_in[i * 3 + 2] = evs[1];
					face_in.push_back(evs[1]); face_in.push_back(v_2); face_in.push_back(evs[0]);
					face_in.push_back(evs[1]); face_in.push_back(evs[0]); face_in.push_back(pinV);
				}
				else{
					face_in[i * 3] = evs[0]; face_in[i * 3 + 1] = v_1; face_in[i * 3 + 2] = evs[1];
					face_in.push_back(pinV); face_in.push_back(evs[0]); face_in.push_back(evs[1]);
					face_in.push_back(pinV); face_in.push_back(evs[1]); face_in.push_back(v_2);
				}
			}
			else if (num_splits == 3)
			{
				face_in[i * 3] = indices[0]; face_in[i * 3 + 1] = evs[0]; face_in[i * 3 + 2] = evs[2];
				face_in.push_back(evs[0]); face_in.push_back(indices[1]); face_in.push_back(evs[1]);
				face_in.push_back(evs[1]); face_in.push_back(indices[2]); face_in.push_back(evs[2]);
				face_in.push_back(evs[0]); face_in.push_back(evs[1]); face_in.push_back(evs[2]);
			}
		}
	}
	noTotalTriangles = face_in.size() / 3;
	noTotalVertex = vertices.size();
	/********************************************************************************************************/
	//incremental displacement
	/********************************************************************************************************/
	MatrixMesh mm(vertices, face_in);
	vertices.clear();
	face_in.clear();
	int steps = 10;
	Vector3f vn, vec, v_step;
	float Gi0, s_Gi0, off_step;
	float d_steps = 1.0 / (float)steps;
	for (int j = 0; j < steps; j++)
	{
#pragma omp parallel for
		for (int vi = 0; vi < mm.nbV; vi++)
		{
			Gi0 = getSubPixel(smooth_gray_invert, texcors[vi][0], texcors[vi][1], tex_w, tex_h);
			s_Gi0 = Gi0 * s_weight;
			off_step = s_Gi0 * d_steps;
			vn = mm.VN(vi);
			vec = vn * off_step;
			v_step = mm.V(vi) + vec;
			mm.set_vV(vi, v_step);
		}
		mm.ComputeNormals();
	}

	texCoords_in.clear();
	vertices_in.clear();
	face_in.clear();
	normals_in.clear();
	for (int i = 0; i < noTotalVertex; i++)
	{
		vertices_in.push_back(mm.V(i)[0]);
		vertices_in.push_back(mm.V(i)[1]);
		vertices_in.push_back(mm.V(i)[2]);
		texCoords_in.push_back(texcors[i][0]);
		texCoords_in.push_back(texcors[i][1]); 
		normals_in.push_back(mm.VN(i)[0]);
		normals_in.push_back(mm.VN(i)[1]);
		normals_in.push_back(mm.VN(i)[2]);
	}
	vertices.clear();
	texcors.clear();
	for (int i = 0; i < noTotalTriangles; i++)
	{
		int v1 = mm.F(i, 0);
		int v2 = mm.F(i, 1);
		int v3 = mm.F(i, 2);
		face_in.push_back(v1);
		face_in.push_back(v2);
		face_in.push_back(v3);

		face_in.push_back(v1);
		face_in.push_back(v2);
		face_in.push_back(v3);

		face_in.push_back(v1);
		face_in.push_back(v2);
		face_in.push_back(v3);

		face_in.push_back(0);
	}
} 
    
 ```
