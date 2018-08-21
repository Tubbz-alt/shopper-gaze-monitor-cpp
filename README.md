# Shopper Gaze Monitor

## Introduction

This shopper gaze monitor application is part of a series of how-to Computer Vision (CV) code sample exercises using Intel® OpenVINO.

## What it is

This application is designed for a retail shelf mounted camera system that counts the the number of passers-by that look towards the display vs. the number of people that pass by the display without looking. It is intended to provide real-world marketing statistics for in-store shelf-space advertising.

## How it works

The application uses a video source such as a camera to grab frames, and then uses 2 different Deep Neural Networks (DNNs) to process the data. The first network looks for faces, and then if successful is counted as a "shopper"

A second neural network is then used to determine the head pose detection for each detected face. If the person's head is facing towards the camera, it is counted as a "looker".

The data can then optionally be sent to an MQTT machine to machine messaging server, as part of a retail data analytics system.

The DNN models used are Intel optimized models that are part of the OpenVINO toolkit.

You can find them here:

- /opt/intel/computer_vision_sdk/deployment_tools/intel_models/face-detection-adas-0001
- /opt/intel/computer_vision_sdk/deployment_tools/intel_models/head-pose-estimation-adas-0001

![Code organization](./images/arch3.png)

The program creates three threads for concurrency:

- main thread that performs the video i/o
- worker thread that processes video frames using the deep neural networks
- worker thread that publishes any MQTT messages

## Setting the build environment

You must configure the environment to use OpenVINO one time per session by the running the following command:

    source /opt/intel/computer_vision_sdk/bin/setupvars.sh

## Building the code

First change directory to the `shopper-gaze-monitor` directory:

    cd shopper-gaze-monitor

If you do not yet have a `build` directory create one:

    mkdir build

Then change to that directory:

    cd build

Now run the following commands:

    cmake ..
    make

Once the commands are finished, you should have built the `monitor` application executable.

## Running the code

To see a list of the various options:

    ./monitor -h

To run the application with the needed models using the webcam:

    ./monitor -m=/opt/intel/computer_vision_sdk/deployment_tools/intel_models/face-detection-adas-0001/FP32/face-detection-adas-0001.bin -c=/opt/intel/computer_vision_sdk/deployment_tools/intel_models/face-detection-adas-0001/FP32/face-detection-adas-0001.xml -pm=/opt/intel/computer_vision_sdk/deployment_tools/intel_models/head-pose-estimation-adas-0001/FP32/head-pose-estimation-adas-0001.bin -pc=/opt/intel/computer_vision_sdk/deployment_tools/intel_models/head-pose-estimation-adas-0001/FP32/head-pose-estimation-adas-0001.xml

### Hardware acceleration

This application can take advantage of the hardware acceleration in OpenVINO by using the `-b` and `-t` parameters.

For example, to use the OpenVINO backend with the GPU in 32-bit mode:

    ./monitor -m=/opt/intel/computer_vision_sdk/deployment_tools/intel_models/face-detection-adas-0001/FP32/face-detection-adas-0001.bin -c=/opt/intel/computer_vision_sdk/deployment_tools/intel_models/face-detection-adas-0001/FP32/face-detection-adas-0001.xml -pm=/opt/intel/computer_vision_sdk/deployment_tools/intel_models/head-pose-estimation-adas-0001/FP32/head-pose-estimation-adas-0001.bin -pc=/opt/intel/computer_vision_sdk/deployment_tools/intel_models/head-pose-estimation-adas-0001/FP32/head-pose-estimation-adas-0001.xml -b=2 -t=1

To run the code using 16-bit floats, you have to both set the `-t` flag to use the GPU in 16-bit mode, as well as use the FP16 version of the Intel models:

    ./monitor -m=/opt/intel/computer_vision_sdk/deployment_tools/intel_models/face-detection-adas-0001/FP16/face-detection-adas-0001.bin -c=/opt/intel/computer_vision_sdk/deployment_tools/intel_models/face-detection-adas-0001/FP16/face-detection-adas-0001.xml -pm=/opt/intel/computer_vision_sdk/deployment_tools/intel_models/head-pose-estimation-adas-0001/FP16/head-pose-estimation-adas-0001.bin -pc=/opt/intel/computer_vision_sdk/deployment_tools/intel_models/head-pose-estimation-adas-0001/FP16/head-pose-estimation-adas-0001.xml -b=2 -t=2

### Machine to machine messaging with MQTT

If you wish to use a MQTT server to publish data, you should set the following environment variables before running the program:

    export MQTT_SERVER=localhost:1883
    export MQTT_CLIENT_ID=cvservice

You should change the `MQTT_CLIENT_ID` to a unique value for each monitoring station, so you can track the data for individual locations. For example:

    export MQTT_CLIENT_ID=shelf1337