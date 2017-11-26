---
layout: post
title:  "extract statistics of point cloud"
date:   2017-09-05 14:25:28
categories: graphics
---
an implementation of computing normals and curvature of point cloud, without any libs, feel free to use.

```C++
//@
// extract statistics of depth surface
//
#pragma once
#include <algorithm>
#include "../Utils/ITMLibDefines.h"

inline bool abs_compare(float a, float b)
{
	return (std::abs(a) < std::abs(b));
}

inline void computeNormal(const std::vector<float> & accu, Vector3f & outNormal, float & outCurvature, Vector4f & xyz_centroid)
{
	std::vector<float> covariance_matrix;
	covariance_matrix.resize(9);
	xyz_centroid[0] = accu[6]; xyz_centroid[1] = accu[7]; xyz_centroid[2] = accu[8];
	xyz_centroid[3] = 1;
	covariance_matrix[0] = accu[0] - accu[6] * accu[6];;
	covariance_matrix[1] = accu[1] - accu[6] * accu[7];
	covariance_matrix[2] = accu[2] - accu[6] * accu[8];
	covariance_matrix[4] = accu[3] - accu[7] * accu[7];
	covariance_matrix[5] = accu[4] - accu[7] * accu[8];
	covariance_matrix[8] = accu[5] - accu[8] * accu[8];
	covariance_matrix[3] = covariance_matrix[1];
	covariance_matrix[6] = covariance_matrix[2];
	covariance_matrix[7] = covariance_matrix[5];

	/////////////////eigen3x3/////////////////
	float eigen_value;
	Vector3f eigen_vector;
	float scale = *std::max_element(covariance_matrix.begin(), covariance_matrix.end(), abs_compare);
	if (scale <= std::numeric_limits < float > ::min())
		scale = 1.0f;
	std::vector<float> scaledMat;
	scaledMat.resize(9);
	for(size_t i = 0; i< 9; i++) scaledMat[i] = covariance_matrix[i] / scale;
	
	/////////////////////compute roots//////////////
	Vector3f eigenvalues;
	float c0 = scaledMat[0] * scaledMat[4] * scaledMat[8]
		+ 2.0f * scaledMat[1] * scaledMat[2] * scaledMat[5]
		- scaledMat[0] * scaledMat[5] * scaledMat[5]
		- scaledMat[4] * scaledMat[2] * scaledMat[2]
		- scaledMat[8] * scaledMat[1] * scaledMat[1];
	float c1 = scaledMat[0] * scaledMat[4] -
		scaledMat[1] * scaledMat[1] +
		scaledMat[0] * scaledMat[8] -
		scaledMat[2] * scaledMat[2] +
		scaledMat[4] * scaledMat[8] -
		scaledMat[5] * scaledMat[5];
	float c2 = scaledMat[0] + scaledMat[4] + scaledMat[8];

	if (fabs(c0) < std::numeric_limits < float > ::epsilon())
	{
		//////compute roots2////////
		eigenvalues[0] = 0.f;
		float d = c2 * c2 - 4.0f * c1;
		if (d < 0.f)
			d = 0.f;
		float sd = std::sqrt(d);
		eigenvalues[2] = 0.5f * (c2 + sd);
		eigenvalues[1] = 0.5f * (c2 - sd);
		//////end compute roots2////////
	}
	else
	{
		const float s_inv3 = 1.0f / 3.0f;
		const float s_sqrt3 = std::sqrt(3.0f);
		float c2_over_3 = c2 * s_inv3;
		float a_over_3 = (c1 - c2 * c2_over_3) * s_inv3;
		if (a_over_3 > 0.f)
			a_over_3 = 0.f;
		float half_b = 0.5f * (c0 + c2_over_3 * (2.0f * c2_over_3 * c2_over_3 - c1));
		float q = half_b * half_b + a_over_3 * a_over_3 * a_over_3;
		if (q > 0.f)
			q = 0.f;
		float rho = std::sqrt(-a_over_3);
		float theta = std::atan2(std::sqrt(-q), half_b) * s_inv3;
		float cos_theta = std::cos(theta);
		float sin_theta = std::sin(theta);
		eigenvalues[0] = c2_over_3 + 2.0f * rho * cos_theta;
		eigenvalues[1] = c2_over_3 - rho * (cos_theta + s_sqrt3 * sin_theta);
		eigenvalues[2] = c2_over_3 - rho * (cos_theta - s_sqrt3 * sin_theta);
		if (eigenvalues[0] >= eigenvalues[1])
			std::swap(eigenvalues[0], eigenvalues[1]);
		if (eigenvalues[1] >= eigenvalues[2])
		{
			std::swap(eigenvalues[1], eigenvalues[2]);
			if (eigenvalues[0] >= eigenvalues[1])
				std::swap(eigenvalues[0], eigenvalues[1]);
		}
		if (eigenvalues[0] <= 0.f)
		{
			//////compute roots2////////
			eigenvalues[0] = 0.f;
			float d = c2 * c2 - 4.0f * c1;
			if (d < 0.f)
				d = 0.f;
			float sd = ::std::sqrt(d);
			eigenvalues[2] = 0.5f * (c2 + sd);
			eigenvalues[1] = 0.5f * (c2 - sd);
			//////end compute roots2////////
		}
	}
	///////////////////////////////////
	eigen_value = eigenvalues[0] * scale;
	scaledMat[0] -= eigenvalues[0];
	scaledMat[4] -= eigenvalues[0];
	scaledMat[8] -= eigenvalues[0];
	Vector3f scaledVec0(scaledMat[0], scaledMat[1], scaledMat[2]);
	Vector3f scaledVec1(scaledMat[3], scaledMat[4], scaledMat[5]);
	Vector3f scaledVec2(scaledMat[6], scaledMat[7], scaledMat[8]);
	Vector3f vec1 = cross(scaledVec0, scaledVec1);
	Vector3f vec2 = cross(scaledVec0, scaledVec2);
	Vector3f vec3 = cross(scaledVec1, scaledVec2);
	float len1 = length(vec1);
	float len2 = length(vec2);
	float len3 = length(vec3);
	if (len1 >= len2 && len1 >= len3)
		eigen_vector = vec1 / len1;
	else if (len2 >= len1 && len2 >= len3)
		eigen_vector = vec2 / len2;
	else
		eigen_vector = vec3 / len3;
	///////////////////////////////end of eigen3x3///////////////
	outNormal.x = eigen_vector[0];
	outNormal.y = eigen_vector[1];
	outNormal.z = eigen_vector[2];
	float eig_sum = covariance_matrix[0] + covariance_matrix[4] + covariance_matrix[8];
	if (eig_sum < std::numeric_limits < float > ::epsilon())
		outCurvature = 0.f;
	else
		outCurvature = fabsf(eigen_value / eig_sum);
}
  ```
