FROM ubuntu:16.04

# libgpr.so => /usr/local/src/github.com/tkyonezu/iroha-pi/docker/rel/iroha/lib/libgpr.so
# libcares.so.2 => /usr/local/src/github.com/tkyonezu/iroha-pi/docker/rel/iroha/lib/libcares.so.2

ARG NUMCORE=${NUMCORE:-4}

RUN apt-get update && apt-get -y upgrade

RUN apt-get -y --no-install-recommends install \
    build-essential python-software-properties \
    automake libtool \
    libssl-dev zlib1g-dev libc6-dbg golang \
    git ssh tar gzip ca-certificates python3 python3-pip python3-setuptools \
    lcov \
    wget curl cmake file unzip gdb iputils-ping vim ccache \
    gcovr vera++ cppcheck doxygen graphviz graphviz-dev \
 && apt-get -y clean

RUN pip3 install --upgrade pip \
 && pip3 install flask requests

# install cmake 3.5.1
RUN git clone https://gitlab.kitware.com/cmake/cmake.git /tmp/cmake \
 && (cd /tmp/cmake; git checkout 64130a7e793483e24c1d68bdd234f81d5edb2d51) \
 && (cd /tmp/cmake; /tmp/cmake/bootstrap --parallel=${NUMCORE} --enable-ccache) \
 && make -j${NUMCORE} -C /tmp/cmake \
 && make -C /tmp/cmake install \
 && rm -fr /tmp/cmake

# o install boost 1.65.1
RUN git clone https://github.com/boostorg/boost /tmp/boost \
 && (cd /tmp/boost; git checkout 436ad1dfcfc7e0246141beddd11c8a4e9c10b146) \
 && (cd /tmp/boost; git submodule init) \
 && (cd /tmp/boost; git submodule update --recursive) \
 && (cd /tmp/boost; /tmp/boost/bootstrap.sh --with-libraries=system,filesystem) \
 && (cd /tmp/boost; /tmp/boost/b2 headers) \
 && (cd /tmp/boost; /tmp/boost/b2 cxxflags="-std=c++14" -j${NUMCORE} install) \
 && rm -fr /tmp/boost

# o install protobuf
RUN git clone https://github.com/google/protobuf /tmp/protobuf \
 && (cd /tmp/protobuf; git checkout 80a37e0782d2d702d52234b62dd4b9ec74fd2c95) \
 && cmake -Dprotobuf_BUILD_TESTS=OFF -Dprotobuf_BUILD_SHARED_LIBS=ON -H/tmp/protobuf/cmake -B/tmp/protobuf/.build \
 && cmake --build /tmp/protobuf/.build --target install -- -j${NUMCORE} \
 && ldconfig \
 && rm -fr /tmp/protobuf

# x install gflags
RUN git clone https://github.com/gflags/gflags /tmp/gflags \
 && (cd /tmp/gflags; git checkout f8a0efe03aa69b3336d8e228b37d4ccb17324b88) \
 && cmake -H/tmp/gflags -B/tmp/gflags/build \
 && cmake --build /tmp/gflags/build --target install -- -j${PARALLELISM} \
 && rm -rf /tmp/gflags

# o install grpc
RUN git clone https://github.com/c-ares/c-ares /tmp/c-ares \
 && (cd /tmp/c-ares; git checkout 3be1924221e1326df520f8498d704a5c4c8d0cce) \
 && cmake -H/tmp/c-ares -B/tmp/c-ares/build \
 && cmake --build /tmp/c-ares/build --target install -- -j${NUMCORE} \
 && ldconfig \
 && rm -fr /tmp/c-ares \
 && git clone https://github.com/grpc/grpc /tmp/grpc \
 && (cd /tmp/grpc; git checkout bfcbad3b86c7912968dc8e64f2121c920dad4dfb) \
 && (cd /tmp/grpc; git submodule update --init third_party/benchmark) \
 && cmake -DgRPC_ZLIB_PROVIDER=package -DgRPC_CARES_PROVIDER=package -DgRPC_SSL_PROVIDER=package \
        -DgRPC_PROTOBUF_PROVIDER=package -DgRPC_GFLAGS_PROVIDER=package -DBUILD_SHARED_LIBS=ON -H/tmp/grpc -B/tmp/grpc/.build \
 && cmake --build /tmp/grpc/.build --target install -- -j${NUMCORE} \
 && ldconfig \
 && rm -fr /tmp/grpc

# x install gtest
RUN git clone https://github.com/google/googletest /tmp/googletest \
 && (cd /tmp/googletest ; git checkout ec44c6c1675c25b9827aacd08c02433cccde7780) \
 && cmake -H/tmp/googletest -B/tmp/googletest/build \
 && cmake --build /tmp/googletest/build --target install -- -j${PARALLELISM} \
 && rm -rf /tmp/googletest

# x install spdlog
RUN git clone https://github.com/gabime/spdlog /tmp/spdlog \
 && (cd /tmp/spdlog ; git checkout f85a08622e20b74bff34381cafcb8ef8167b29d0) \
 && cmake -DSPDLOG_BUILD_TESTING=OFF -H/tmp/spdlog -B/tmp/spdlog/build \
 && cmake --build /tmp/spdlog/build --target install \
 && rm -rf /tmp/spdlog

# x install rxcpp
RUN git clone https://github.com/Reactive-Extensions/RxCpp /tmp/RxCpp \
 && (cd /tmp/RxCpp; git checkout 1b2e0589f19cb34d8cd58803677701dcf2161876) \
 && cmake -H/tmp/RxCpp -B/tmp/RxCpp/build \
 && cmake --build /tmp/RxCpp/build --target install \
 && rm -fr /tmp/RxCpp

# x install rapidjson
RUN git clone https://github.com/miloyip/rapidjson /tmp/rapidjson \
 && (cd /tmp/rapidjson; git checkout f54b0e47a08782a6131cc3d60f94d038fa6e0a51) \
 && cmake -DRAPIDJSON_BUILD_EXAMPLES=OFF -H/tmp/rapidjson -B/tmp/rapidjson/build \
 && cmake --build /tmp/rapidjson/build --target install \
 && rm -fr /tmp/rapidjson

# x install optional
RUN git clone https://github.com/martinmoene/optional-lite /tmp/optional-lite \
 && (cd /tmp/optional-lite ; git checkout a0ddabb8b52e1eaaf0dd1515bb85698b747528e4) \
 && cp -r /tmp/optional-lite/include/nonstd /usr/local/include \
 && rm -fr /tmp/optional-lite

# o install libpq
RUN git clone https://git.postgresql.org/git/postgresql.git /tmp/postgresql \
 && (cd /tmp/postgresql; git checkout 029386ccbddd0a33d481b94e511f5219b03e6636) \
 && (cd /tmp/postgresql; /tmp/postgresql/configure --without-readline --prefix=/usr/local) \
 && make -j${NUMCORE} -C /tmp/postgresql/src/bin/pg_config \
 && make -j${NUMCORE} -C /tmp/postgresql/src/interfaces/libpq \
 && make -C /tmp/postgresql/src/bin/pg_config install \
 && make -C /tmp/postgresql/src/interfaces/libpq install \
 && make -C /tmp/postgresql/src/include install; \
    rm -fr /tmp/postgresql

# x install pqxx
RUN git clone https://github.com/jtv/libpqxx /tmp/libpqxx \
 && (cd /tmp/libpqxx; git checkout 5b17abce5ac2b1a2f8278718405b7ade8bb30ae9) \
 && (cd /tmp/libpqxx; /tmp/libpqxx/configure --disable-documentation --with-pic) \
 && make -j${NUMCORE} -C /tmp/libpqxx \
 && make -C /tmp/libpqxx install \
 && rm -fr /tmp/libpqxx

# x install cpp_redis 4.3.0
RUN git clone https://github.com/Cylix/cpp_redis /tmp/cpp_redis \
 && (cd /tmp/cpp_redis; git checkout f390eef447a62dcb6da288fb1e91f25f8a9b838c) \
 && (cd /tmp/cpp_redis; git submodule update --init --recursive) \
 && cmake -H/tmp/cpp_redis -B/tmp/cpp_redis/build \
 && cmake --build /tmp/cpp_redis/build --target install -- -j${NUMCORE} \
 && rm -fr /tmp/cpp_redis

# o install tbb
RUN git clone https://github.com/01org/tbb /tmp/tbb \
 && (cd /tmp/tbb; git checkout eb6336ad29450f2a64af5123ca1b9429ff6bc11d) \
 && make -j${NUMCORE} -C /tmp/tbb tbb_build_prefix=build \
 && cp /tmp/tbb/build/build_debug/*.so* /usr/local/lib \
 && cp /tmp/tbb/build/build_release/*.so* /usr/local/lib \
 && cp -r /tmp/tbb/include/* /usr/local/include \
 && ldconfig \
 && rm -fr /tmp/tbb

# o install ed25519
RUN git clone git://github.com/warchant/ed25519.git /tmp/ed25519 \
 && (cd /tmp/ed25519; git checkout 56d926c4489aa7b2bb664a48294c5ee2d6e1c283) \
 && cmake -DCMAKE_BUILD_TYPE=Debug -DTESTING=OFF -H/tmp/ed25519 -B/tmp/ed25519/build \
 && cmake --build /tmp/ed25519/build --target install -- -j${NUMCORE} \
 && ldconfig \
 && rm -fr /tmp/ed25519

WORKDIR /opt/iroha

CMD ["/bin/bash"]
