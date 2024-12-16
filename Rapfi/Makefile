# Directories
BUILD_DIR_SINGLE := build-single
BUILD_DIR_MULTI := build-multi
OUTPUT_DIR := wasm-output

# Common flags
COMMON_FLAGS := -DCMAKE_BUILD_TYPE=Release \
                -DNO_COMMAND_MODULES=ON \
                -DUSE_SSE=OFF \
                -DUSE_AVX2=OFF \
                -DUSE_AVX512=OFF \
                -DUSE_BMI2=OFF \
                -DUSE_VNNI=OFF \
                -Wno-dev

# Multi-threaded specific flags
EMSCRIPTEN_FLAGS := -s USE_PTHREADS=1 \
                    -s PTHREAD_POOL_SIZE=4 \
                    -s MODULARIZE=1 \
                    -s EXPORT_NAME="createRapfiModule" \
                    -s PROXY_TO_PTHREAD=1

# Targets
.PHONY: all single multi clean

all: single multi

single:
	@echo "Building single-threaded version..."
	mkdir -p $(BUILD_DIR_SINGLE)
	cd $(BUILD_DIR_SINGLE) && emcmake cmake .. $(COMMON_FLAGS) -DNO_MULTI_THREADING=ON
	cd $(BUILD_DIR_SINGLE) && emmake make
	mkdir -p $(OUTPUT_DIR)
	cp $(BUILD_DIR_SINGLE)/*.wasm $(BUILD_DIR_SINGLE)/*.js $(BUILD_DIR_SINGLE)/*.data $(OUTPUT_DIR)/

multi:
	@echo "Building multi-threaded version..."
	mkdir -p $(BUILD_DIR_MULTI)
	cd $(BUILD_DIR_MULTI) && emcmake cmake .. $(COMMON_FLAGS) -DNO_MULTI_THREADING=OFF
	cd $(BUILD_DIR_MULTI) && emmake make CXXFLAGS="$(EMSCRIPTEN_FLAGS)" LDFLAGS="$(EMSCRIPTEN_FLAGS)"
	mkdir -p $(OUTPUT_DIR)
	# Copy files if they exist
	cp $(BUILD_DIR_MULTI)/*.wasm $(BUILD_DIR_MULTI)/*.js $(BUILD_DIR_MULTI)/*.data $(OUTPUT_DIR)/
	@if [ -e $(BUILD_DIR_MULTI)/*.worker.js ]; then \
	  cp $(BUILD_DIR_MULTI)/*.worker.js $(OUTPUT_DIR)/; \
	else \
	  echo "Warning: No worker file found. Ensure PROXY_TO_PTHREAD is properly set in your configuration."; \
	fi

clean:
	@echo "Cleaning build directories and output..."
	rm -rf $(BUILD_DIR_SINGLE) $(BUILD_DIR_MULTI) $(OUTPUT_DIR)