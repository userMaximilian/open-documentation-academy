# Introduction

This file contains all of the images associated with specific Ubuntu WSL documentation pages.

You will use this file to suggest alternative (alt) text for images in the documentation.
Alt text helps people who are visually-impaired or have a visual disability to interpret images using screen reader technology.
A lack of alt text or low-quality alt text is an accessibility issue that can make documentation difficult or impossible to navigate.

If you are struggling to create alt text for an image it might be
useful to go to the documentation and view the image in context.
The live docs that include these images can be found below:

[Official Ubuntu WSL docs](https://canonical-ubuntu-wsl.readthedocs-hosted.com/en/latest/)

# Image accessibility and alt text

Images in this file are defined using conventional markdown syntax.
When you edit this file, images will be prefixed with `!` followed by a pair of square brackets and a pair of parentheses. The alt text will be found in square brackets `[here]` and the url to the image will be in parentheses `(here)`:

```markdown
![this is alt text](url)
```

The square brackets in the Ubuntu WSL docs currently contain image dimensions, such as `[|624x489]`, which is not valid alt text.

Any alt text should fit the following criteria:

- Describe the content of the image clearly and in as few words as possible
- Preferably be one sentence in length with no or minimal punctuation
- Avoid repetition and include only the essential information once
- Avoid unecessary words, such as "image of..." (the user will know it's an image)

It is difficult to create good alt text for certain images.
An example is an image of a terminal that contains a lot of text.
You would not reproduce all of that text but instead attempt to
explain what it is and what it does:

> Bash snippet showing the updating of packages

> Log of output showing successful running of script

# Table of contents

- [Tutorials](#tutorials)
    - [Working with Visual Studio Code](#working-with-visual-studio-code) 
    - [Windows and Ubuntu Interoperability](#windows-and-ubuntu-interoperability)
    - [Run a .NET Echo Bot as a systemd service on Ubuntu WSL](#run-a-.NET-Echo-Bot-as-a-systemd-service-on-Ubuntu-WSL)
    - [Enabling GPU acceleration with the NVIDIA CUDA platform](#enabling-GPU-acceleration-with-the-NVIDIA-CUDA-platform)
    - [Use WSL for data science and engineering](#use-WSL-for-data-science-and-engineering)
- [Howtos](#howtos)
    - [Install Ubuntu on WSL2](#install-Ubuntu-on-WSL2)

# Tutorials

## Working with Visual Studio Code

[Link to full documentation page](https://canonical-ubuntu-wsl.readthedocs-hosted.com/en/latest/tutorials/vscode/) 

![|624x483](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/vscode/msstore.png?raw=true)

As an example, alt text for the image above could be: `Install page for Visual Studio Code on the Microsoft Store`.

![|624x353](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/vscode/download-vs-code.png?raw=true)

![|624x492](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/vscode/aditional-tasks.png?raw=true)

![|624x469](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/vscode/remote-extension.png?raw=true)

![|624x321](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/vscode/downloading-vscode-server.png?raw=true)

![|624x351](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/vscode/hello-world.png?raw=true)

## Windows and Ubuntu Interoperability

[Link to full documentation page](https://canonical-ubuntu-wsl.readthedocs-hosted.com/en/latest/tutorials/interop/)

![|624x347](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/interop/jupyter.png?raw=true)

![Jupyter Notebook: A Beginner's Tutorial|624x251](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/interop/jupyter-python.jpg?raw=true)

![|624x469](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/interop/jupyter-script.png?raw=true)

![|624x352](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/interop/ubuntu-home.png?raw=true)

![|624x389](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/interop/spreadsheet.png?raw=true)

## Run a .NET Echo Bot as a systemd service on Ubuntu WSL

[link to documentation page](https://canonical-ubuntu-wsl.readthedocs-hosted.com/en/latest/tutorials/dotnet-systemd/)

![|624x153](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/dotnet-systemd/templates.png?raw=true)

![|624x357](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/dotnet-systemd/welcome-to-dotnet.png?raw=true)

![|624x347](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/dotnet-systemd/your-bot-is-ready.png?raw=true)

![|624x411](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/dotnet-systemd/bot-framework-emulator.png?raw=true)

![|624x357](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/dotnet-systemd/ipconfig.png?raw=true)

![|624x411](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/dotnet-systemd/emulator-settings.png?raw=true)

![|624x411](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/dotnet-systemd/open-a-bot.png?raw=true)

![|624x411](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/dotnet-systemd/start-chatting.png?raw=true)

![|624x381](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/dotnet-systemd/program-cs.png?raw=true)

![|624x77](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/dotnet-systemd/systemctl-status-inactive.png?raw=true)

![|624x401](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/dotnet-systemd/systemctl-status-running.png?raw=true)

![|624x411](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/dotnet-systemd/start-chatting-service.png?raw=true)

## Enabling GPU acceleration with the NVIDIA CUDA platform

[link to documentation page](https://canonical-ubuntu-wsl.readthedocs-hosted.com/en/latest/tutorials/gpu-cuda/#)

![|624x283](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/gpu-cuda/install-drivers.png?raw=true)

![|624x136](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/gpu-cuda/downloads-folder.png?raw=true)

![|358x312](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/gpu-cuda/nvidia-allow-changes.png?raw=true)

![|369x175](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/gpu-cuda/default-dir.png?raw=true)

![|368x168](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/gpu-cuda/please-wait-install.png?raw=true)

![|541x276](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/gpu-cuda/splash-screen.png?raw=true)

![|542x412](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/gpu-cuda/license.png?raw=true)

![|536x397](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/gpu-cuda/installation-options.png?raw=true)

![|542x404](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/gpu-cuda/installing.png?raw=true)

![|544x406](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/gpu-cuda/install-finished.png?raw=true)

![|544x559](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/gpu-cuda/done-done.png?raw=true)

![|624x135](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/gpu-cuda/make.png?raw=true)

![|548x599](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/gpu-cuda/device-query.png?raw=true)

## Use WSL for data science and engineering

[link to documentation page](https://canonical-ubuntu-wsl.readthedocs-hosted.com/en/latest/tutorials/data-science-and-engineering/)

![|624x373](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/data-science-engineering/octave.png?raw=true)

![|570x225](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/data-science-engineering/save-file.png?raw=true)

![|579x540](https://github.com/ubuntu/WSL/blob/main/docs/tutorials/assets/data-science-engineering/julia-fractal.png?raw=true)

# Howtos

## Install Ubuntu on WSL2

[Link to documentation page](https://canonical-ubuntu-wsl.readthedocs-hosted.com/en/latest/guides/install-ubuntu-wsl2/)

![|624x489](https://github.com/ubuntu/WSL/blob/main/docs/guides/assets/install-ubuntu-wsl2/choose-distribution.png?raw=true)

![|624x580](https://github.com/ubuntu/WSL/blob/main/docs/guides/assets/install-ubuntu-wsl2/search-ubuntu-windows.png?raw=true)
