# Copyright 2015, Google Inc.
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are
# met:
#
#     * Redistributions of source code must retain the above copyright
# notice, this list of conditions and the following disclaimer.
#     * Redistributions in binary form must reproduce the above
# copyright notice, this list of conditions and the following disclaimer
# in the documentation and/or other materials provided with the
# distribution.
#     * Neither the name of Google Inc. nor the names of its
# contributors may be used to endorse or promote products derived from
# this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
# "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
# LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
# A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
# OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
# SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
# LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
# DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
# THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

# Dockerfile for the gRPC Java dev image
FROM grpc/java_base

# Required by accessing Android's aapt
RUN apt-get install -y lib32stdc++6 lib32z1 && apt-get clean

# Install Android SDK 24.2
RUN curl -L http://dl.google.com/android/android-sdk_r24.2-linux.tgz | tar xz -C /usr/local

# Environment variables
ENV ANDROID_HOME /usr/local/android-sdk-linux
ENV PATH $PATH:$ANDROID_HOME/tools
ENV PATH $PATH:$ANDROID_HOME/platform-tools
# Some old Docker versions consider '/' as HOME
ENV HOME /root

# Update sdk for android 5.1 (API level 22)
RUN echo y | android update sdk --all --filter platform-tools,build-tools-22.0.1,sys-img-armeabi-v7a-addon-google_apis-google-22,sys-img-armeabi-v7a-addon-google_apis-google-21,sys-img-armeabi-v7a-android-19,addon-google_apis-google-22,addon-google_apis-google-21,addon-google_apis-google-19,extra-android-m2repository,extra-google-m2repository --no-ui --force


# Create an AVD with API level 22
RUN echo no | android create avd --force -n avd-google-api-22 -t "Google Inc.:Google APIs:22" --abi google_apis/armeabi-v7a
RUN echo no | android create avd --force -n avd-google-api-21 -t "Google Inc.:Google APIs:21" --abi google_apis/armeabi-v7a
RUN echo no | android create avd --force -n avd-google-api-19 -t "Google Inc.:Google APIs:19" --abi default/armeabi-v7a

# Pull gRPC Java and trigger download of needed Maven and Gradle artifacts.
RUN git clone --depth 1 https://github.com/grpc/grpc-java.git /var/local/git/grpc-java && \
  cd /var/local/git/grpc-java && \
  ./gradlew grpc-core:install grpc-stub:install grpc-okhttp:install grpc-protobuf-nano:install && \
  rm -r "$(pwd)"

# Pull gRPC Android integration test App
RUN git clone --depth 1 https://github.com/madongfly/grpc-android-test.git /var/local/git/grpc-android-test

# Config android sdk for gradle
RUN cd /var/local/git/grpc-android-test && echo "sdk.dir=/usr/local/android-sdk-linux" > local.properties

# Build apks to trigger download of needed Maven and Gradle artifacts.
RUN cd /var/local/git/grpc-android-test && ./gradlew assembleDebug
RUN cd /var/local/git/grpc-android-test && ./gradlew assembleDebugAndroidTest
