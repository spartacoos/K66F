# Overview

As an IoT developer, you might think of machine learning as a server-side technology. In the traditional view, sensors on your device capture data and send it to the cloud, where Machine Learning (ML) models on hefty machines make sense of it. A network connection is obligatory, and you are going to expect some latency, not to mention hosting costs.

But more and more, developers want to deploy their ML models to the edge, on IoT devices themselves. If you bring ML closer to your sensors, you remove your reliance on a network connection, and you can achieve much lower latency without a round trip to the server.

This is especially exciting for IoT, because less network utilization means lower power consumption. Also, you can better guarantee the security and privacy of your users, since you do not need to send data back to the cloud unless you know for sure that it is relevant.

In the following guide, you will learn how you can perform machine learning inference on an Arm Cortex-M microcontroller with TensorFlow Lite for Microcontrollers.

**About TensorFlow Lite**

TensorFlow Lite is a set of tools for running machine learning models on-device. TensorFlow Lite powers billions of mobile app installs, including Google Photos, Gmail, and devices made by Nest and Google Home.

With the launch of TensorFlow Lite for Microcontrollers, developers can run machine learning inference on extremely low-powered devices, like the Cortex-M microcontroller series. Watch the video to learn more about the announcement:

[![An introduction to TensorFlow Lite](http://img.youtube.com/vi/DKosV_-4pdQ/0.jpg)](http://www.youtube.com/watch?v=DKosV_-4pdQ)

# Getting Started

**Before you begin** 
Here is what you will need to complete the guide:

* Computer that supports Mbed CLI (version 1.10.4)

* NXP FRDM K66F

* Mini-USB cable

* Python 2.7. Using pyenv is recommended to manage python versions.

For Windows users, install Ubuntu 20.04 LTS in a VirtualBox. Refer to the following videos to set up:

**How to install VirtalBox 6.0.10 on Windows 10**

![IMAGE ALT TEXT HERE](http://img.youtube.com/vi/8mns5yqMfZk/0.jpg)

[Watch here](http://www.youtube.com/watch?v=8mns5yqMfZk)

**How to install Ubuntu 20.04 on VirtualBox in Windows 10**

![IMAGE ALT TEXT HERE](http://img.youtube.com/vi/x5MhydijWmc/0.jpg)

[Watch here](http://www.youtube.com/watch?v=x5MhydijWmc)


**Getting started**

TensorFlow Lite for Microcontrollers supports several devices out of the box, and is relatively easy to extend to new devices. For this guide, we will focus on the **NXP FRDM K66F**.

![IMAGE ALT TEXT HERE](https://www.nxp.com/assets/images/en/dev-board-image/FRDM-K66F-BD.jpg)


We will deploy a sample application that uses the microphone on the K66F and a TensorFlow machine learning model to detect the words “yes” and “no”.

To do this, we will show you how to complete the following steps:

1.  Download and build the sample application
    

2.  Deploy the sample to your K66F
    

3.  Use new trained models to recognize different words

# Download and build the sample application

**Install Arm toolchain and Mbed CLI**
-   Download [Arm cross compilation](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads) toolchain. Select the correct toolchain for the OS that your computer is running. For Windows users, if you have already set up the Linux virtual environment, install the toolchain there.
    
-   To build and deploy the application, we will use the [Mbed CLI](https://github.com/ARMmbed/mbed-cli). We recommend that you install Mbed CLI with our installer. If you need more customization, you can perform a manual install. Although this is not recommended.  
      
    If you do not already have Mbed CLI installed, download the installer:  
      
    [Mac installer](https://github.com/ARMmbed/mbed-cli-osx-installer/releases/tag/v0.0.10)
-   After Mbed CLI is installed, tell Mbed where to find the Arm embedded toolchain.  
  ```    
 mbed config -G GCC_ARM_PATH <path_to_your_arm_toolchain>/bin
 ```

**Important:** We recommend running the following commands from inside the Mbed CLI terminal that gets launched with the Mbed CLI Application. This is because it will be much quicker to set up, because it resolves all your environment dependencies automatically.


# Build and compile micro speech example

Navigate to the directory where you keep code projects. Run the following command to download TensorFlow Lite source code.

```git clone https://github.com/tensorflow/tensorflow.git```

While you wait for the project to download, let’s explore the project files on [GitHub](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/lite/micro/examples/micro_speech) and learn how this TensorFlow Lite for Microcontrollers example works.

The code samples audio from the microphone on the K66F. The audio is run through a Fast Fourier transform to create a spectrogram. The spectrogram is then fed into a pre-trained machine learning model. The model uses a [convolutional neural network](https://developers.google.com/machine-learning/practica/image-classification/convolutional-neural-networks) to identify whether the sample represents either the command “yes” or “no”, silence, or an unknown input. We will explore how this works in more detail later in the guide.

Here are descriptions of some interesting source files:

-   nxp_k66f[/audio_provider.cc](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/lite/micro/examples/micro_speech/nxp_k66f/audio_provider.cc) captures audio from the microphone on the device.
    
-   [micro_features/micro_features_generator.cc](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/lite/micro/examples/micro_speech/micro_features/micro_features_generator.cc): uses a Fast Fourier transform to create a spectrogram from audio.
    
-   [micro_features/model.cc](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/lite/micro/examples/micro_speech/micro_features/model.cc). This file is the machine learning model itself, represented by a large array of unsigned char values.
    
-   [command_responder.cc](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/lite/micro/examples/micro_speech/command_responder.cc) is called every time a potential command has been identified.
    
-   [main.cc](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/lite/micro/examples/micro_speech/main.cc). This file is the entry point for the Mbed program, which runs the machine learning model using TensorFlow Lite for Microcontrollers.
    

After the project has downloaded, you can run the following commands to navigate into the project directory and build it:

```cd tensorflow```

  
```make -f tensorflow/lite/micro/tools/make/Makefile TARGET=mbed TAGS="nxp_k66f" generate_micro_speech_mbed_project```

This will create a folder in ```tensorflow/lite/micro/tools/make/gen/mbed_cortex-m4/prj/micro_speech/mbed```  that contains the source and header files, Mbed driver files, and a README.

```cd tensorflow/lite/micro/tools/make/gen/mbed_cortex-m4/prj/micro_speech/mbed```

  Execute the following while remembering to use Python 2.7 (you can do this by using a virtual environment with pip):
```mbed config root . ``` 

```mbed deploy```

```mbed compile -m K66F -t GCC_ARM```

For some Mbed compilers, you may get compile error in mbed_rtc_time.cpp. Go to  mbed-os/platform/mbed_rtc_time.h  and comment line 32 and line 37:

```
//#if !defined(__GNUC__) || defined(__CC_ARM) || defined(__clang__) struct timeval { 
time_t  tv_sec; int32_t tv_usec; 
};
//#endif
```

If your system does not recognize the board with the mbed detect command. Follow the instructions for setting up [DAPLink](https://armmbed.github.io/DAPLink/?board=FRDM-K66F) for the [K66F](https://os.mbed.com/platforms/FRDM-K66F/).

Connect the USB cable to the micro USB port. When the Ethernet port is facing towards you, the micro USB port is left of the Ethernet port.

Now, we are ready to flash the device:

```mbed compile -m K66F -t GCC_ARM –flash```

Disconnect USB cable from the device to power down the device and connect back the power cable to start running the model.

Connect to serial port with baud rate of 9600 and correct serial device to view the output from the MCU. In linux, you can run the following screen command if the serial device is  /dev/ttyACM0:

```sudo screen /dev/ttyACM0 9600```

Saying "Yes" will print "Yes" and "No" will print "No" on the serial port.

# Project structure
While the project builds, we can look in more detail at how it works.

### Convolutional neural networks
Convolutional networks are a type of deep neural network. These networks are designed to identify features in multidimensional vectors. The information in these vectors is contained in the relationships between groups of adjacent values.

These networks are usually used to analyze images. An image is a good example of the multidimensional vectors described above, in which a group of adjacent pixels might represent a shape, a pattern, or a texture. During training, a convolutional network can identify these features and learn what they represent. The network can learn how simple image features, like lines or edges, fit together into more complex features, like an eye, or an ear. The network can also learn, how those features are combined to form an input image, like a photo of a human face. This means that a convolutional network can learn to distinguish between different classes of input image, for example a photo of a person and a photo of a dog.

While they are often applied to images, which are 2D grids of pixels, a convolutional network can be used with any multidimensional vector input. In the example we are building in this guide, a convolutional network has been trained on a spectrogram that represents 1 second of audio bucketed into multiple frequencies.

The following image is a visual representation of the audio. The network in our sample has learned which features in this image come together to represent a "yes", and which come together to represent a "no".

![enter image description here](https://hackster.imgix.net/uploads/attachments/1039035/yesnofeatures_PcVyip5SvM.png?auto=compress,format&w=680&h=510&fit=max)

To generate this spectrogram, we use an interesting technique that is described in the next section.

# Feature generation with Fast Fourier transform
In our code, each spectrogram is represented as a 2D array, with 43 columns and 49 rows. Each row represents a 30ms sample of audio that is split into 43 frequency buckets.

To create each row, we run a 30ms slice of audio input through a Fast Fourier transform. Fast Fourier transform analyzes the frequency distribution of audio in the sample and creates an array of 256 frequency buckets, each with a value from 0 to 255. These buckets are averaged together into groups of 6, leaving us with 43 buckets. The code in the file [micro_features/micro_features_generator.cc](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/lite/micro/examples/micro_speech/micro_features/micro_features_generator.cc) performs this action.

To build the entire 2D array, we combine the results of running the Fast Fourier transform on 49 consecutive 30ms slices of audio, with each slice overlapping the last by 10ms. The following diagram should make this clearer:
![enter image description here](https://hackster.imgix.net/uploads/attachments/1038841/pasted_image_0.png?auto=compress,format&w=680&h=510&fit=max)

You can see how the 30ms sample window is moved forward by 20ms each time until it has covered the full one-second sample. The resulting spectrogram is passed into the convolutional model. 

# Recognition and windowing
The process of capturing one second of audio and converting it into a spectrogram leaves us with something that our ML model can interpret. The model outputs a probability score for each category it understands (yes, no, unknown, and silence). The probability score indicates whether the audio is likely to belong to that category.

The model was trained on one-second samples of audio. In the training data, the word “yes” or “no” is spoken at the start of the sample, and the entire word is contained within that one-second. However, when this code is running, there is no guarantee that a user will begin speaking at the very beginning of our one-second sample.

If the user starts saying “yes” at the end of the sample instead of the beginning, the model might not be able to understand the word. This is because the model uses the position of the features within the sample to help predict which word was spoken.

To solve this problem, our code runs inference as often as it can, depending on the speed of the device, and averages all of the results within a rolling 1000ms window. The code in the file [recognize_commands.cc](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/lite/micro/examples/micro_speech/recognize_commands.cc) performs this action. When the average for a given category in a set of predictions goes above the threshold, as defined in [recognize_commands.h](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/lite/micro/examples/micro_speech/recognize_commands.h), we can assume a valid result.

# Interpreting the results

The RespondToCommand method in [command_responder.cc](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/lite/micro/examples/micro_speech/command_responder.cc) is called when a command has been recognized. Currently, this results in a line being printed to the serial port. Later in this guide, we will modify the code to display the result on the screen.

# Deploy the example to your K66F

In the previous section of this guide, we explained the build process for a keyword spotting example application.

Now that the build has completed, we will look in this section of the guide at how to deploy the binary to the K66F and test to see if it works.

First, plug in your K66F board via USB. The board should show up on your machine as a USB mass storage device. Copy the binary file that we built earlier to the USB storage.

Note: if you have skipped the previous steps, download the [binary file]() to proceed.

Use the following command:

```cp ./BUILD/K66F/GCC_ARM/mbed.bin /Volumes/K66F/```

Depending on your platform, the exact copy command and paths may vary. When you have copied the file, the LEDs on the board should start flashing, and the board will eventually reboot with the sample program running.

# Test keyword spotting

The program outputs recognition results to its serial port. To see the output of the program, we will need to establish a serial connection with the board at 9600 baud.

The board’s USB UART shows up as  ```/dev/tty.usbmodemXXXXXXX```.We can use ‘screen’ to access the serial console. Although, ‘screen’ is not installed on Linux by default, you can use apt-get install screen to install the package.

Run the following command in a separate terminal:

```screen /dev/tty.usbmodemXXXXXX 9600```

**Note**: this command may very depending on where your board is plugged.

Try saying the word “yes” several times. You should see some output like the following:

Heard yes (208) @116448ms  
Heard unknown (241) @117984ms  
Heard no (201) @124992ms

The LED  will blink green if "Heard yes!", as you can see in the following image:

<INSERT IMAGE OF LED BLINKING HERE>[]()

Congratulations! You are now running a machine learning model that can recognize keywords on an Arm Cortex-M7 microcontroller, directly on your K66F.

It is easy to change the behavior of our program, but is it difficult to modify the machine learning model itself? The answer is no, and the next section of this guide, [Retrain the machine learning model](https://developer.arm.com/solutions/machine-learning-on-arm/developer-material/how-to-guides/build-arm-cortex-m-voice-assistant-with-google-tensorflow-lite/retrain-the-machine-learning-model), will show you how.

# Retrain the machine learning model

The model that we are using for speech recognition was trained on a dataset of one-second spoken commands called the [Speech Commands Dataset](https://ai.googleblog.com/2017/08/launching-speech-commands-dataset.html). The dataset includes examples of the following ten different words:

yes, no, up, down, left, right, on, off, stop, go

While the model in this sample was originally trained to recognize “yes” and “no”, the TensorFlow Lite for Microcontrollers source contains scripts that make it easy to retrain the model to classify any other combination of these words.

We are going to use another pre-trained model to recognize “up” and “down”, instead. If you are interested in the full flow including the training of the model refer to the [Supplementary information: model training](https://developer.arm.com/solutions/machine-learning-on-arm/developer-material/how-to-guides/build-arm-cortex-m-voice-assistant-with-google-tensorflow-lite/supplementary-information-model-training) section of this guide.

To build our new ML application we will now follow these steps:

-   Download a pretrained model that has been trained and frozen using TensorFlow.
    
-   Look at how the TensorFlow model gets converted to the TensorFlow Lite format.
    
-   Convert the TensorFlow Lite model into a C source file.
    
-   Modify the code and deploy to the device.
    

Note: Building TensorFlow and training the model will each take a couple of hours on an average computer. We will not perform this at this stage. For a full guide on how to do this, refer to the [Supplementary information: model training](https://developer.arm.com/solutions/machine-learning-on-arm/developer-material/how-to-guides/build-arm-cortex-m-voice-assistant-with-google-tensorflow-lite/supplementary-information-model-training) section in this guide.

# Convert the model

Starting from the trained model to obtain a converted model that can run on the controller itself, we need to run a conversion script: the [TensorFlow Lite converter](https://www.tensorflow.org/lite/convert). This tool uses clever tricks to make our model as small and efficient as possible, and to convert it to a TensorFlow Lite FlatBuffer. To reduce the size of the model, we used a technique called [quantization](https://www.tensorflow.org/lite/performance/post_training_quantization). All weights and activations in the model get converted from 32-bit floating point format to an 8-bit and fixed-point format, as you can see in the following command:

![Convert the model to the TensorFlow Lite format code image](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAABAAAAAAwCAIAAAAASp9qAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAHsoSURBVHhe7d0FvNRV2gfwmRuExAXpUFoJBUQJRUHEQhQLW1TsrlVfY6011u5E18K1JQQxwMZGQEzAAFG6G26935kzjMPcS4jo4u488hn//zPnnKd+T5ypG92lz+mfD312+7atv/vuu8gmQznRSFEkUlScuP1NVLZcWY8rlq8It7+JypUrF41GV6xYUVSE/9qoQoUKhYWFy5cvT9yvgXJycsqUKbNy5cqCgoLE0G+hsmXL1qxZc9q0aRuwnCKEzM/Pp05iKBLJzs4mz9oVbNSo0Ztvvtmla5dFixbNnzc/MboGwmWzzTaz27JlyxJDqwgv48XFCS+6zcvLmz9//jptG83OrlSxwsIFCxP3kUhWdlZubm5hQWGqHbCuWLEiXZg3MWRmVlblypVJzjuJoTjrKlWqzJ07NylMjLKyquRVTlOQt8rm5i5ZsiRxvwbKLZOLEb7FvxGjZG7cuPHVV1+9cOHCM844YzV5MpShDGUoQxnKUIb+FMoK/9vUGpGC4g3s/pHWf8O6f6Sh18ius0NFesR1dv9Iw7p06dIN6/6R7nbKlCkbtpxPFy9enNr9I23xOhV0Zpg9e3b+yvx1dv8IF6Yo2f0jvFJx5VYLvj62LS4sTO3+UVFhEZ+m2cHmGv3U7h/Z3xkjtftHbufMmZMOcjNLKFiwcuU6u3/EOOT5rd0/cqK7/vrrncEeeOCBdHkylKEMZShDGcpQhv4U2kTfAchQhv5bKbwjlLjJUIYylKEMZShDGfrTaRN9ByBDGfpvpUz3n6EMZShDGcpQhv6zlN2gTfsZE76qV7fO3LlzE2N/GEWj0TJlypT8EEhWVlbZsmWzs7PTPrmRStbm5OSUPKjk5uYWrRoMn/9emb8yUuI446nwkfTE/SoyXr58eRfJp2Jc/G+DTkSEjMmzOhd8cVnPI1alSpU233zzpUuXbgpHMmJvttlmXBP77E00mle5crly5Sjo9q9yYuQRkMjPz9+IAleoUIFZmIKj03z9X0+pkGDSvLw8dkC/x8LsCfPheynwLx5tmIaxihUrNmnSZPny5Rt8fCJ21apVybmWJJOhtRCn1KtXT/jz1J8f/oAXB1o57gvcwaZx48bwwKdhjkgPc34/Gm27sUJbyatSpYqyshboAmeDBg1wTPvE5l+LJFuKePyvD7EkGmMFII4TsGnUqBHgJb28UdC40bOW+LVhiOLEUAmSgaHRBV3+/Ejf6ASQwRExVxUXKyWhxDBp8F0gHvydJWaDKVS3ZcuWJVPZn0bZDdu0nz7hqzq1a23cA0DNmjWlvFSQVatWrVWrVttuu60az8oo8URc/44dO1oyZcqUxNDqZMJWW23VsmXLvCpVNMfBQ3DMam3atKlQseLSJUsM2vnEE0/84osvUl2o/1a3sG7cpInrZat/Il+u79q1K5TPnj3b8oYNG7Zu3XrzatUK4l/whQl869atW3sVLVq0iJPEf/369ZUfPkvqWL16dZO32Wab3DJlrCWDPsm05s2bb7HFFiYYXGcYH3bYYT179vzkk0+CCiXN+KcRm2+//fadO3cm/48//sh0Rxx+eNu2bWntdlMoVAyusspl/MIppQYPnx577LETJk7Y4K+FpFE0K6t79+7M0qFDB5hZsGBBahL5E4jWVKbsn5+ahUO7du0CJCZPnrwiP//oo44SgJ06dZowYYLATMz7jdQrTl9++eXSZcsOPPDAHXbYQTaw/+LFixMzIhEsHnvssTFjxhgPI9K66PCYGuxrob322uuAAw74/vvv58+PffFDOhKe1v6hZpRe/gQufw41bdr0yiuvlHX5OjV7bzDpjNmHZVITcqmkYEvggCe7zpw5c8mSJVa1b7/Dgw8+OHHixEmTJrmNZmdvFDTus88++++///jx4xcujH0NKYTbemKsVGrRosURRxwhf/7www9u9YW6urTmQ9G544475s2bZ846rbHJEjftscceObk5s+fM3oDvR/1++j3hps7KCS5KrSOpRE2tyE477QSNSoA0hZ1aef/994Oi4siz0HjUkUcql9AIohuMxj333FNKhIqNlbVE8ZFHHgmBYsSt+qjQg1wqGjVpt9xyiyiTLddpjU2H9PT6AYqkRpBBXdmOO+4oxDRvaO+99+YU5XvatGliPGnJjh07PPzwv7766islJjlYq1YtkftHdzslq9vvpPVvErKyIlH/W8s8za4qm7hZb+rbt++uu+5qbbiV60899VT145hjjrn++uuPPvro3DK54SmUl5cncYioxH28uieu4te77777TTfddMkll9x37737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xfseZOSjGvUqHnFsq4uvxCvIlvrvjQpVcfy6actwO6F5a7evl630yOyFTy1awZJW9Zs25/nlxR/WqEepykcdP6J83pBImeV77ssOj81djCO9ivY96F+zFrgm80vRMluccR71b//up5+364i7nQcUZD9dkH13fvZdBdlfxNqVBDWLFHXKLt583XkiFjKJq9VpPbP074y4VNICim6nPg36+pTgUlmvU2xZVzFy1P/mm2+cDw8//PDEE7+bSrJ2QE09fyZJ2q9Xr17qbyhtGMnAUroMrEhVXP2vUJdKpVqs1MFUql279rnnnut4HPu1zdatL7roojUZOfwNKL6bO3fOHrvv3q1bNztrqGTvpUuX1qhe/bBDD6X18uXLp0+f7vSF9AaOScrNsmXLHN3DYJ06dZwZNqw4blzKiRTFOsu0FyyrVKmyyy67hDP6W2+99fbbb4dxI9vvsAOvjBgx4sMPP1yyZInmjCYep02b9u9//1sNzquSd/HFF8+YPkMntGLFiieffHLcuHHWasTVM0Vu9OjRgwYNMl9T7sTPOgz6wQcfvPbaa2xUtWoVFTqwQwsWLKhWrXqTJk0crUKPsnZf2urEE0+0VTgskj+8wHnbbbfp6Y855hi1XGVdtOqnx1NJXdynZ09OcqINv/G05ZZbduzY8fHHHx/y8svOoA61Dm2l/vwTuNP30UcfPe200xJDv4+0NVdddVVIE5jOi//ev2PGpZdeOmHCBA0xA4Y/GNSyVcsjjzxy8aLFnnVUCD9hDpHaCL4zyOzvv/++Wyco5x92aN68OU/169dvTa+S6pDCX4OaNGnSI488oqmi/umnnw7lRrbZZhvyDBgwwAQdWPg9Hxj49NNPhwwZQmZnMHNs/sILL4wfP577DjvssEcefeSXn38RHieccMLdd989Y8YMYhx88MHatV9++YXp7Cxa8vJiL1wFys8vEEs87qD8xBNP0CV+Hms5fPjrZcqUhUb4oawE8UX8R98JSUEed1p46qmnvv3227CPoOWdIPPYsWNJSzwK6iDh5LHHHps7b65O69hjj124YKHzm4XyDr1Abuc4cTpQhd8dD3umkiXUB/X/+7//Swz9boI9+aVdu3awqmxQUA8KXfTiO5o6C916662mQTUJxRevQQWoE/vvl//9p8k/sTbfPf300+GHhIG/f//+OnU7w8Pnn38O0lAhIixn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K0Szo7MeX7zix4K6V1ZaMGTF0s9X1jylYnFR1tKPlxfF/0bfsgkFudWz6l5dednYwiWfLK+0Z/kydbOmXLSgaFmib8vZPLtg7hq//1GrVq2//e1vbdu2ffvttxs0aLDNNttcddVVb7311pp+5MdRYdy4cQ888EBqu6/BrRD/Xar0VQ45q7/REY49ae9+aBON/J63ROxQtmzZX79KGw5X69rQMUZLvWLFipJngL322uu2229rvW3r5FdnTC75NZq6deuecuopdrj+uusTQylUu3Zthnqw34NvvPHmyg36yRreue66615++eWBAwe6rVy58kknnZSXl3fd9dfHfgNnQ83IX6mesq3W/+mnnz788MMnrPqRY55iUnsuj//p2TCISuViMO2c8Mgjj4wePZr6q0EiK2vL+vV/+umnxG2GMvQ/T47IZaIRB/rsSHHmuJyhTYEk98JIVIFfWRzJfOwnQxnaiJQ4AJT8FaCcvKyq+5crXFE897llQrDcljmbH7lZ+Va5c59auuDt5bUvqpg8ABTMjMx/dWnR0ljLVVwUydt9s6qHlJty0fyixUW5dXO3uClv2tWLln4b6zirHVqx8u5lZt69ZMkXpTegHTt2PP/889566+3+/fvXr1//2muvef/9D4YMHXLRRReVLRP7awBavbvvvvu1115zNvj73//uUZ89c+bMH3/88bTTTtP5DR06NLzCfeKJJ/7www8uypUrt/feex955JGWv/rqq/369Vu0aJHmcvfddy9TpkyVKlUGDx78XPxvxzZv3vyEE07YZZdd5s6de/vtt7/33nul/rxX7Tq1n+z/JHPhvnTp0n/84x8jR45s0qSJ04g++KOPPurSpYvjh/HJkyf36tXrwAMP1Nk/88wzzz//PHannnrqBx988Morr2y55ZYXXnjhk/9+ctSoUXfcfkezZs1sfvHFF2tVQ5960EEHHXvssTVq1DDSfffu27TaRtO//fbbn3zyyZ06dXJGuvPOO+lYrVq1Pn369OzZMxweRn02ai0HAJ305ptvbhozsoazFo5nn3P2lJ+mtG7d+sYbb2SBqVOnlmypd9ttt8suu4wNFyxY8Msvv5D/qaeecgDo0aPH/Pnz6cXyzz777LRp0xo3bsy8bP7zzz+zNoOs6VfSdthhB+x23HFHXfjVV1/92WefhXEnGbuFAwDWnTt3dg4k+RtvvHH//feHXxBjvd69e++7775z5sz517/+9e677zr13XDDDYzQqlWrxYsXU/b111/fdtttL7/88pYtW8LJjBkzJk2axP7h7HTFFVcceuih1McrzjZDGcpQhjKUoQxl6H+CYi+g+l/aR4BipJ/3TPgXp8L5xUvH5pffJje3zmofucmpE62wXdkKO5Qrv1XZaHY0u1rWyu8LipYWFxdGVv6cXzivKKd64rWkoiVFkWjE+JqoUqVKixYt1g5qoCd+993UadPz8vIqV6pct07sz3Zo0zVw559/vhZ2/Pjx11xzzbfffvvmm2/q5PTrujp97Xnnnff3v1+mB01+gtzCs846y/wnn3xSv3vEEUdkZ2cb1EF++umnevedd945/HkO3aRm+tprrzXuKEKYsEMalStbrkGDBlWrVr3nnnt0n5pOgz9Onox1vXr1tPv2vO+++3T/bdq02X///R0kCKkh7tatm/64uLioU6eOtWrV0sTrm+fPm7982fKbbrrpkksuJjOmwRee0pvq72+77bbUn9HVzpqgF7fDKaecQhfCH3zwwW+99Ra+WufEvDWQ+dpl1jj33HMp+/XXXztg7LH7Hrm5udQvKiqiUVr3j4zrzh0PLHzhhRc068mmWds9ZswYOmrTdd5G9OVbb731I488MmvWrGOOOabmmv9qvXOd5Zdccsm8efOcl0oBYZz12LFj//a3v40YMcIZJvljiw4e2223nWOVs99hhx3GI45zzifISUyv71Bkcwixs4OEwwOcMGbyzQGnrIoVK27wB6IylKEMZShDGcpQhv6ilJUdjXXnaR+ZKJ2KI4s+XFFcVFxhmzLZFX59f7jsljkVdy5Tadey5dvlRnMiWVmRouQLvsWR4pXF4WsGaP5ryyafNW9Z/N2AUikrK0tbFj7PU1RYmL9yZRBMa6jVQ/fcc3e5crH+e/ny5frCuXPn/vLLL6NHj9bnhbb1m2+++eqrr5OtpPZuu7Zt3d51113PP//8hx9+2LRp0/Llyxv55JOP+/fvP2jQIFvp5k1etmxZs2bNevXq5ZihS9YKh03SyFp09913vfjii3fccbuOs2HDhkUFBRMnTnRuefnll6+77rrhw4dTZJtWraZMmfLggw/++9///jn+N5KpM2bM2Dp16rp2xiCtLpnkGv2vv/4m9YNM7du3p+nNN99MQsuDdltssYVDhU7XIy3C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)
This conversion will not only reduce the size of the network, but will avoid floating point computations that are more computationally expensive.

To save time, we will skip this step and instead download the [tiny_conv.tflite](https://developer.arm.com/-/media/Files/downloads/Machine%20learning%20how-to%20guides/tiny_conv.tflite?revision=495eb362-4325-49b8-b3ba-3141df0c9b95&la=en&hash=0F37BA2C5DE95A1561979CDD18973171767A47C3).

The final step in the process is to convert this model into a C file that we can drop into our Mbed project.

To do this conversion, we will use a tool called xxd. Issue the following command:

```xxd -i  tiny_conv.tflite > ../micro_features/model.cc```

Next, we need to update model.cc so that it is compatible with our code. First, open the file. The top two lines should look similar to the following code, although the exact variable name and hex values may be different:
```
const  unsigned  char  g_model[] DATA_ALIGN_ATTRIBUTE = {  
0x18, 0x00, 0x00, 0x00, 0x54, 0x46, 0x4c, 0x33, 0x00, 0x00, 0x0e, 0x00,
```
You need to add the include from the following snippet and change the variable declaration without changing the hex values:
```
#include "tensorflow/lite/micro/examples/micro_speech/micro_features/model.h"  
const unsigned char g_tiny_conv_micro_features_model_data[] = {  
0x18, 0x00, 0x00, 0x00, 0x54, 0x46, 0x4c, 0x33, 0x00, 0x00, 0x0e, 0x00,
```
Next, go to the very bottom of the file and find the unsigned int variable.
```
unsigned int tiny_conv_tflite_len = 18216;
```
Change the declaration to the following code, but do not change the number assigned to it, even if your number is different from the one in this guide.
```
const int g_tiny_conv_micro_features_model_data_len = 18216;
```
Finally, save the file, then copy the ```tiny_conv_micro_features_model_data.cc```file into the ```tensorflow/tensorflow/lite/micro/tools/make/gen/mbed_cortex-m4/prj/micro_speech/mbed/tensorflow/lite/micro/examples/micro_speech/micro_features``` directory.

# Modify the device code

If you build and run your code now, your device should respond to the words “up” and “down”. However, the code was written to assume that the words are “yes” and “no”. Let’s update the references and the user interface so that the appropriate words are printed.

First, go to the following directory:

```tensorflow/lite/micro/examples/micro_speech/```

and open the file:

```micro_features/micro_model_settings.cc```

You will see the following category labels:
```
const char* kCategoryLabels[kCategoryCount] = {  
"silence",  
"unknown",  
"yes",  
"no",  
};
```
The code uses this array to map the output of the model to the correct value. Because we specified our wanted_words as “up, down”in the training script, we should update this array to reflect these words in the same order. Edit the code so it appears as follows:
```
const char* kCategoryLabels[kCategoryCount] = {  
"silence",  
"unknown",  
"up",  
"down",  
};
```
Next, we will update the code in command_responder.cc to reflect these new labels, modifying the if statements and the DisplayStringAt call:
```
void RespondToCommand(tflite::ErrorReporter* error_reporter,  
int32_t current_time, const char* found_command,  
uint8_t score, bool is_new_command) {  
if (is_new_command) {  
error_reporter->Report("Heard %s (%d) @%dms", found_command, score,  
current_time);  
if(strcmp(found_command, "up") == 0) {  
lcd.Clear(0xFF0F9D58);  
lcd.DisplayStringAt(0, LINE(5), (uint8_t *)"Heard up", CENTER_MODE);  
} else if(strcmp(found_command, "down") == 0) {  
lcd.Clear(0xFFDB4437);  
lcd.DisplayStringAt(0, LINE(5), (uint8_t *)"Heard down", CENTER_MODE);  
} else if(strcmp(found_command, "unknown") == 0) {  
lcd.Clear(0xFFF4B400);  
lcd.DisplayStringAt(0, LINE(5), (uint8_t *)"Heard unknown", CENTER_MODE);  
} else {  
lcd.Clear(0xFF4285F4);  
lcd.DisplayStringAt(0, LINE(5), (uint8_t *)"Heard silence", CENTER_MODE);  
}  
}  
}
```
Now that we have updated the code, go back to the mbed directory:
```
cd <path_to_tensorflow>/tensorflow/lite/micro/tools/make/gen/mbed_cortex-m4/prj/micro_speech/mbed
```
and run the following command to rebuild the project:

```mbed compile -m K66F -t GCC_ARM```

Finally, copy the binary to the USB storage of the device, using the same method that you used earlier. You should now be able to say “up” and “down” to update the display.

# Troubleshooting

We have found some common errors that users face and have listed them here to help you get started with your application as quickly as possible.
If you encounter:

```Mbed CLI issues or Error: collect2: error: ld returned 1 exit status```

Purge the cache with the following command:

```mbed cache purge```

You probably also have a stale BUILD folder. Clean up your directory and try again:

```rm -rf BUILD```

Error: Prompt wrapping around line

If your terminal is wrapping your text as show here:

![Error prompt wrapping around line image](data:image/png;base64,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)

In your terminal type:

```export PS1='\u@\h: '```

For a more minimalist type:

```export PS1='> '```

Error: "Requires make version 3.82 or later (current is 3.81)"

If you encounter this error, install the brew and make by typing the following code:

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

``` 
brew install make
```

**Note**: On a Mac, you might have to use gmake instead of make, to run your commands.

Error: -bash: mbed: command not found

If you encounter this error, try the following fixes.

For Mac:

We recommend using the [installer](https://github.com/ARMmbed/mbed-cli-osx-installer/releases/tag/v0.0.10) and running the downloaded Mbed CLI App. This app will automatically launch a shell with all the dependencies solved for you.

If installed manually, make sure to follow these [instructions](https://os.mbed.com/docs/mbed-os/v5.12/tools/macos.html).
