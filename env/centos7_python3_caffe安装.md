# centos python3 caffe 安装

### 基础包
```
yum -y install leveldb-devel snappy-devel opencv-devel hdf5-devel gflags-devel glog-devel lmdb-devel
yum -y install openblas-devel python36-devel
```

### boost安装
```
wget https://dl.bintray.com/boostorg/release/1.67.0/source/boost_1_67_0.tar.gz
tar zxvf boost_1_67_0.tar.gz
./bootstrap.sh --with-toolset=gcc
./b2 cflags='-fPIC' cxxflags='-fPIC' include=/usr/include/python3.6m
./b2 install
ln -s /usr/local/lib/libboost_python36.so /usr/lib64/libboost_python3.so
echo /usr/local/lib >> /etc/ld.so.conf.d/caffe.conf
ldconfig
```


### protobuf 安装
```
wget https://github.com/google/protobuf/releases/download/v3.5.1/protobuf-cpp-3.5.1.tar.gz
wget https://github.com/google/protobuf/releases/download/v3.5.1/protobuf-python-3.5.1.tar.gz
tar zxvf protobuf-cpp-3.5.1.tar.gz
tar zxvf protobuf-python-3.5.1.tar.gz
./configure && make && make install
cd python
python setup.py build
python setup.py install
ln -s /usr/local/lib/libprotobuf.so.13.0.0 /usr/lib64/libprotobuf.so.13
```

### caffe 安装
```
git clone --depth 1 https://github.com/BVLC/caffe.git
cp Makefile.config.example Makefile.config
protoc src/caffe/proto/caffe.proto --cpp_out=.
make -j32 && make pycaffe -j32
cp -r python/caffe/ /usr/local/python3/lib/python3.6/site-packages
cp .build_release/lib/* /usr/lib64
```