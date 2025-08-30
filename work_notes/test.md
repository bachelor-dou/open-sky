docker build -t manylinux2_28-py310-cann8_1_x86_64 \
  --build-arg ARCH=x86_64 \
  --build-arg PYTHON_VERSION=310 \
  -f Dockerfile .
