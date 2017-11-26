---
layout: post
title:  "continue: uniform and adaptive Loop-color-displacement subdivision"
date:   2017-08-28 15:25:28
categories: graphics
---
This program is an implementation of matrix representation of mesh tolopogy, hope useful to geometry community.

```C++
//////////////////////////////////////////////////////////////////////////////
//MatrixMesh structure
class MatrixMesh
		{
		protected:
			vector<Vector3f> vertices;
			vector<int> faces;
			vector< vector<int> > vertex_to_faces; // vertex_to_faces[i][j] is the j-th neighboring face of the vertex i
			vector< vector<int> > face_to_faces; // face_to_faces[i][j] is the j-th neighboring face of the face i (only 3)
			vector< Vector3f> FNormals;
			vector< Vector3f> VNormals;
		public:
			int nbV; // number of vertices
			int nbF; // number of faces
			MatrixMesh() {}
			MatrixMesh(const vector<Vector3f>& _verts, const vector<int>& _faces);
			~MatrixMesh();
			void InitNeighboringData(); // initialize all neighboring information
			void ComputeNormals(); // compute all normals
			Vector3f V(int v) const
			{
				return vertices[v];
			}
			int F(int v, int j) const
			{
				return faces[v * 3 + j];
			}
			Vector3f VN(int v) const
			{
				return VNormals[v];
			}
			Vector3f FN(int fi) const
			{
				return FNormals[fi];
			}
			void set_vV(int v, const Vector3f & p)
			{
				vertices[v] = p;
			}
			int vfNbNeighbors(int v) const {
				return vertex_to_faces[v].size();
			}
			int vfNeighbor(int v, int i) const {
				return vertex_to_faces[v][i];
			}
		};
    
              MatrixMesh::MatrixMesh(const vector<Vector3f>& _verts, const vector<int>& _faces)
		{
			vertices = _verts;
			faces = _faces;
			nbV = vertices.size();
			nbF = faces.size() / 3;
			InitNeighboringData();
			ComputeNormals();
		}

	      MatrixMesh::MatrixMesh() {
		}

		void MatrixMesh::InitNeighboringData()
		{
			vertex_to_faces.clear();
			vertex_to_faces.resize(nbV);

			int v0, v1, v2, fj;
			for (int fi = 0; fi < nbF; fi++) {
				for (int j = 0; j < 3; ++j) {
					vertex_to_faces[F(fi, j)].push_back(fi);
				}
			}
			face_to_faces.clear();
			face_to_faces.resize(nbF);
			for (int fi = 0; fi < nbF; ++fi) {
				for (int i = 0; i < 3; ++i) {
					v0 = F(fi, i);
					v1 = F(fi, (i + 1) % 3);
					for (unsigned int j = 0; j < vertex_to_faces[v0].size(); ++j) {
						fj = vertex_to_faces[v0][j];
						vector<int> l = vertex_to_faces[v1];
						if (fj != fi && std::find(l.begin(), l.end(), fj) != l.end()) {
							face_to_faces[fi].push_back(fj);
							break;
						}
					}
				}
			}
		}

		void MatrixMesh::ComputeNormals()
		{
			FNormals.clear();
			FNormals.resize(nbF);
#pragma omp parallel for
			for (int fi = 0; fi < nbF; ++fi) {
				const Vector3f & p0 = V(F(fi, 0));
				const Vector3f & p1 = V(F(fi, 1));
				const Vector3f & p2 = V(F(fi, 2));
				FNormals[fi] = cross(p1 - p0, p2 - p0).normalised();
			}
			VNormals.clear();
			VNormals.resize(nbV);
#pragma omp parallel for
			for (int vi = 0; vi < nbV; ++vi) {
				Vector3f n(0.f, 0.f, 0.f);
				for (int j = 0; j < vfNbNeighbors(vi); ++j) {
					n += FNormals[vfNeighbor(vi, j)];
				}
				VNormals[vi] = n.normalised();
			}
		}
    
   ```


this program computes color gradients, while is different to grey space version. It does in 3-channels, resulting in much visible edges, and following the subpixel read funtion, inspired by OpenCV's

```C++
    //color_gradient_domain
    void get_color_gradients(const std::string &filename, const std::string &out_filename, cv::Mat & magnitude, cv::Mat & mag_x, cv::Mat & mag_y, int & tex_w, int & tex_h)
{
	cv::Mat src = cv::imread(filename.c_str(), CV_LOAD_IMAGE_ANYCOLOR | CV_LOAD_IMAGE_ANYDEPTH);
	cv::Size size = src.size();
	tex_w = size.width;
	tex_h = size.height;
	cv::Mat sobel_dismap(size, CV_32F); //derivative magnitude
	cv::Mat sobel_3dx; // per-channel horizontal derivative
	cv::Mat sobel_3dy; // per-channel vertical derivative
	cv::Mat sobel_dx(size, CV_32F);      // maximum horizontal derivative
	cv::Mat sobel_dy(size, CV_32F);      // maximum vertical derivative
	cv::Mat smoothed;
	// Compute horizontal and vertical image derivatives on all color channels separately
	static const int KERNEL_SIZE = 7;
	// For some reason cvSmooth/cv::GaussianBlur, cvSobel/cv::Sobel have different defaults for border handling...
	GaussianBlur(src, smoothed, cv::Size(KERNEL_SIZE, KERNEL_SIZE), 0, 0, cv::BORDER_REPLICATE);
	Sobel(smoothed, sobel_3dx, CV_16S, 1, 0, 3, 1.0, 0.0, cv::BORDER_REPLICATE);
	Sobel(smoothed, sobel_3dy, CV_16S, 0, 1, 3, 1.0, 0.0, cv::BORDER_REPLICATE);
	cv::Mat smoothed_f;
	smoothed.convertTo(smoothed_f, CV_32FC3, 1.0 / 255.0);
	short * ptrx = (short *)sobel_3dx.data;
	short * ptry = (short *)sobel_3dy.data;
	float * ptr0x = (float *)sobel_dx.data;
	float * ptr0y = (float *)sobel_dy.data;
	float * ptr_dis = (float *)sobel_dismap.data;

	const int length_dis = static_cast<const int>(sobel_dismap.step1());
	const int length1 = static_cast<const int>(sobel_3dx.step1());
	const int length2 = static_cast<const int>(sobel_3dy.step1());
	const int length3 = static_cast<const int>(sobel_dx.step1());
	const int length4 = static_cast<const int>(sobel_dy.step1());
	const int length0 = sobel_3dy.cols * 3;
	// B G R 变化梯度最大的通道作为结果
	for (int r = 0; r < sobel_3dy.rows; ++r)
	{
		int ind = 0;

		for (int i = 0; i < length0; i += 3)
		{
			// Use the gradient orientation of the channel whose magnitude is largest
			int mag1 = ptrx[i + 0] * ptrx[i + 0] + ptry[i + 0] * ptry[i + 0];
			int mag2 = ptrx[i + 1] * ptrx[i + 1] + ptry[i + 1] * ptry[i + 1];
			int mag3 = ptrx[i + 2] * ptrx[i + 2] + ptry[i + 2] * ptry[i + 2];
			if (mag1 >= mag2 && mag1 >= mag3)
			{
				ptr0x[ind] = ptrx[i];
				ptr0y[ind] = ptry[i];
			}
			else if (mag2 >= mag1 && mag2 >= mag3)
			{
				ptr0x[ind] = ptrx[i + 1];
				ptr0y[ind] = ptry[i + 1];
			}
			else
			{
				ptr0x[ind] = ptrx[i + 2];
				ptr0y[ind] = ptry[i + 2];
			}
			//BGRA (dir vector, magnitude)
			float x = ptr0x[ind]; // horizontal
			float y = ptr0y[ind]; //vertical
			ptr_dis[ind] = sqrtf(x*x + y*y);
			++ind;
		}
		ptrx += length1;
		ptr_dis += length_dis;
		ptry += length2;
		ptr0x += length3;
		ptr0y += length4;
	}
	cv::Mat adjMap;
	double min, max;
	cv::Mat adjMap_x;
	double min_x, max_x;
	cv::Mat adjMap_y;
	double min_y, max_y;
	// watch out, as convertion with it own max, mag_x and mag_y might be larger than magnitude
	cv::minMaxIdx(sobel_dismap, &min, &max);
	convertScaleAbs(sobel_dismap, adjMap, 255.0 / max);
	sobel_dismap *= 1.0f / max;
	magnitude = sobel_dismap;
	cv::minMaxIdx(sobel_dx, &min_x, &max_x);
	//convertScaleAbs(sobel_dx, adjMap_x, 255.0 / max_x);
	sobel_dx *= 1.0f / max_x;
	mag_x = sobel_dx;
	cv::minMaxIdx(sobel_dy, &min_y, &max_y);
	//convertScaleAbs(sobel_dy, adjMap_y, 255.0 / max_y);
	sobel_dy *= 1.0f / max_y;
	mag_y = sobel_dy;
	cv::imwrite(out_filename.c_str(), adjMap);
}

double getSubPixel(const cv::Mat& dis_map, float& texcoord_x, float& texcoord_y, int& tex_width, int& tex_height)
{
	assert(!dis_map.empty());
	float fx = texcoord_x * tex_width;
	float fy = (1 - texcoord_y) * tex_height;
	//float fy = texcoord_y * tex_height;
	cv::Point2f pt(fx, fy);

	int x = (int)pt.x;
	int y = (int)pt.y;

	int x0 = cv::borderInterpolate(x, dis_map.cols, cv::BORDER_REFLECT_101);
	int x1 = cv::borderInterpolate(x + 1, dis_map.cols, cv::BORDER_REFLECT_101);
	int y0 = cv::borderInterpolate(y, dis_map.rows, cv::BORDER_REFLECT_101);
	int y1 = cv::borderInterpolate(y + 1, dis_map.rows, cv::BORDER_REFLECT_101);

	float det_x = pt.x - (float)x;
	float det_y = pt.y - (float)y;
	uchar intensity = (uchar)((dis_map.at<uchar>(y0, x0) * (1.f - det_x) + dis_map.at<uchar>(y0, x1) * det_x) * (1.f - det_y)
		+ (dis_map.at<uchar>(y1, x0) * (1.f - det_x) + dis_map.at<uchar>(y1, x1) * det_x) * det_y);
	//double intensity = (double)((dis_map.at<float>(y0, x0) * (1.f - det_x) + dis_map.at<float>(y0, x1) * det_x) * (1.f - det_y)
		//+ (dis_map.at<float>(y1, x0) * (1.f - det_x) + dis_map.at<float>(y1, x1) * det_x) * det_y);
	float value = (float)intensity / 255.0f;
	return value;
}

 ```
