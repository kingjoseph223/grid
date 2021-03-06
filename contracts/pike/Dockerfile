# Copyright 2018 Cargill Incorporated
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM hyperledger/grid-dev as grid-pike-builder

COPY contracts/pike /build/contracts/pike
COPY sdk/ /build/sdk/

# Do a release build to cache dependencies
WORKDIR /build/contracts/pike

# Build the contract
ARG REPO_VERSION
RUN sed -i -e "0,/version.*$/ s/version.*$/version\ =\ \"${REPO_VERSION}\"/" Cargo.toml
RUN cargo build --target wasm32-unknown-unknown --release

# Build a scar file
# Copy the packaging directory
COPY contracts/pike/packaging/scar/* \
     /build/contracts/pike/packaging/scar/

# Copy the contract to the packaging directory
RUN cp /build/target/wasm32-unknown-unknown/release/grid-pike-tp.wasm \
    packaging/scar

WORKDIR /build/contracts/pike/packaging/scar

# Create .scar file
RUN tar -jcvf /tmp/grid-pike_${REPO_VERSION}.scar .

# -------------=== pike submitter build ===-------------

FROM hyperledger/sawtooth-sabre-cli

COPY --from=grid-pike-builder \
     /build/target/wasm32-unknown-unknown/release/grid-pike-tp.wasm /tmp

# Copy the contract definition
COPY contracts/pike/pike.yaml /tmp

COPY --from=grid-pike-builder \
    /tmp/grid-pike*.scar /tmp
