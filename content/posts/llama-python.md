---
title: "Python script for Llama 2 conversations"
date: 2023-07-26T18:00:23+02:00
description: Simple Python script that let's you talk to your local Llama2 LLM using Windows.
draft: false
tags: [python, programming, llama, llm, windows, ai]
---

## Introduction

I have played around a bit with the new Llama 2 LLM, more specifically with the 13B parameter Huggingface version that you can download [here](https://huggingface.co/TheBloke/Llama-2-13B-chat-GGML/resolve/main/llama-2-13b-chat.ggmlv3.q4_0.bin). In order to run it, check out [Llama.cpp](https://github.com/ggerganov/llama.cpp). It has precise setup instructions, so I will assume you get that running on your own. What I did not enjoy was having to type long commands into my Windows cmd every time, so I decided to write a short Python script that runs the process in the background and displays the output in the Python shell in real-time (well, almost). Windows paths are weird, especially when you try to put them in a Python string. Double-escaping \\ did not seem to work, so I had to go with raw strings instead. Let's take a look at the script I am using:

## Python
```py
import signal
import subprocess
from sys import exit


def signal_handler(sig, frame):
    exit(0)


def execute_command(command):
    # run command as global subprocess (that can be stopped at any time with signal_handler)
    process = subprocess.Popen(command, shell=True, stdout=subprocess.PIPE, stderr=subprocess.STDOUT, text=True)

    # read from process and print real-time output to python shell
    first_line = True
    while True:
        output = process.stdout.readline()
        if not output and process.poll() is not None:
            break
        # dont print the first line (it includes the prompt itself)
        if first_line:
            first_line = False
        else:
            print(output.strip())

    # wait until process finishes
    return_code = process.wait()
    print("--------------------------------------------------------------------------------")
    return return_code


def main():
    signal.signal(signal.SIGINT, signal_handler)
    while True:
        prompt = input("Prompt: ")
        print("--------------------------------------------------------------------------------")

        command = (r'"C:\Users\myusername\Documents\llama.cpp\build\bin\Release\main.exe" ' +
        r'-m "C:\Users\myusername\Documents\llama.cpp\build\bin\Release\models\llama-2-13b-chat.ggmlv3.q4_0.bin" ' +
        f'--color -p "{prompt}" 2> $null')

        execute_command(command)


main()
```

## Impressions

Even though I compiled llama.cpp with CUDA support (CUDA is installed on my system too), running the model brings my CPU utilization to 50% (i9 12900k) while my GPU utilization is at 1% (RTX 4070). So I assume the specific model I have chosen either does not support CUDA or there is something wrong with my setup. Either way, the model output is approximately as fast as I am reading text, which is fine for now. Even though my Python code includes a signal handler that allows you to terminate the execution with Ctrl+C, I advise you against using this for now. The subprocess won't cleanly shut down and re-running the script can be problematic. I might update the script in the future to scan running processes and shut down any active instances of Llama's main.exe but for now I am happy with the script. Here is an example output:
![targets](/images/python_llama_example.png "Example prompt")

## Outlook

I am glad we have half-decent language models that can run on consumer hardware. Considering how fast these models evolve and how the entry barriers for small models get lower, I can't wait to see what will be available in a few years!

