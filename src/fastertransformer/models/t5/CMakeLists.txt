# Copyright (c) 2019-2023, NVIDIA CORPORATION.  All rights reserved.
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

cmake_minimum_required(VERSION 3.8)

add_library(T5Common STATIC T5AdapterWeight.cc)
set_property(TARGET T5Common PROPERTY POSITION_INDEPENDENT_CODE  ON)
set_property(TARGET T5Common PROPERTY CUDA_RESOLVE_DEVICE_SYMBOLS  ON)
target_link_libraries(T5Common PUBLIC -lcudart LinearAdapterLayer)

add_library(T5Decoder STATIC T5Decoder.cc T5DecoderLayerWeight.cc)
set_property(TARGET T5Decoder PROPERTY POSITION_INDEPENDENT_CODE  ON)
set_property(TARGET T5Decoder PROPERTY CUDA_RESOLVE_DEVICE_SYMBOLS  ON)
target_link_libraries(T5Decoder PUBLIC -lcudart cublasMMWrapper T5Common TensorParallelDecoderSelfAttentionLayer
        TensorParallelDecoderCrossAttentionLayer TensorParallelReluFfnLayer TensorParallelSiluFfnLayer
        layernorm_kernels add_residual_kernels nccl_utils memory_utils tensor cuda_utils logger)

add_library(T5Decoding STATIC T5Decoding.cc T5DecodingWeight.cc)
set_property(TARGET T5Decoding PROPERTY POSITION_INDEPENDENT_CODE  ON)
set_property(TARGET T5Decoding PROPERTY CUDA_RESOLVE_DEVICE_SYMBOLS  ON)
target_link_libraries(T5Decoding PUBLIC -lcudart cublasMMWrapper T5Decoder bert_preprocess_kernels
                                        decoding_kernels DynamicDecodeLayer BaseBeamSearchLayer 
                                        beam_search_topk_kernels gpt_kernels tensor cuda_utils logger)

add_library(T5Encoder STATIC T5Encoder.cc T5EncoderWeight.cc T5EncoderLayerWeight.cc)
set_property(TARGET T5Encoder PROPERTY POSITION_INDEPENDENT_CODE  ON)
set_property(TARGET T5Encoder PROPERTY CUDA_RESOLVE_DEVICE_SYMBOLS  ON)
target_link_libraries(T5Encoder PUBLIC -lcudart bert_preprocess_kernels cublasMMWrapper T5Common
        TensorParallelUnfusedAttentionLayer FusedAttentionLayer TensorParallelReluFfnLayer
        TensorParallelGeluFfnLayer TensorParallelSiluFfnLayer layernorm_kernels add_residual_kernels
        nccl_utils tensor cuda_utils logger)

add_executable(t5_gemm t5_gemm.cc)
target_link_libraries(t5_gemm PUBLIC -lcudart t5_gemm_func memory_utils cuda_utils logger)
