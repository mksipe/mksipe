<p style="text-align: center; font-size: 40px;"> CaeCrack</p>

---
## Project Documentation
- [Project Documentation](#project-documentation)
    - [System Documentation](#system-documentation)
        - [Requirements](#requirements)
            - [Operating System](#operating-system)
            - [Architechture](#architechture)
            - [Network Access](#network-access)
            - [Dependencies](#dependencies)
        - [Design Descisions](#design-descisions)
    - [User Documentation](#user-documentation)
        - [Installation](#installation)
            - [Manual](#manual)
            - [Automatic](#automatic)
        - [Usage](#usage)

### System  Documentation

#### Requirements

##### Operating System

Any operating system that supports rust will run this program properly. 

##### Architechture

The architecture used to create and run caecrack has been on x86_64 based systems. However, Cargo tailors it to your system precisely.

##### Network Access

This program uses crates.io to carry out package retrieval and uses this to compile third-party crates (packages) and requires crates.io to be accessible to work correctly.

##### Dependencies

On the specified system to deploy this tool, you must have [Cargo](https://crates.io/) and rustc installed. Depending on your package manager and sources configured with your package manager, you should be able to install it through the specified dependency. i.e 

```sh
apt install cargo # debian 
pacman -syu cargo # arch
```

#### Design Descisions

The design for this software was to be specifically tailored to two situations, inline and mass ciphers. Although it is unlikely to come across, it is still a feature. Most of the technical decisions were how to allocate and brute force a cipher precisely; other than that, the program is very straightforward.

### User Documentation

#### Installation


To install, there is more than one way to do so. The official way supported by rust is to run:

`cargo install`

This will install the program to the user locally. However, you can make it public by moving the executable to the `/usr/bin/` directory.


##### Manual

To make a manual installation, you need to download the [software](https://github.com/mksipe/caecrack) via git or an HTTP download.

***
>###### git

 ```sh
 git clone https://github.com/mksipe/caecrack
 cd can crack
 ```
***
>###### http

Click the green code button, 
select Download ZIP

```
cd /home/$USER/Downloads
tar xvf caecrack-main.zip
cd caecrack
```
***

With Cargo already installed, all you need to do is use Cargo to run the program.

`cargo run -- <args>`

##### Automatic

An automatic installation will install the program to your system and place it in the /usr/bin directory
> Install.ps1 does not work!

After installing the program, all that is required is to do the following:

> change executable permissions

`chmod 700 install.sh`

> make install

`sudo ./install.sh`

After the install, the source directory may be deleted.

#### Usage

By default, the following are built into the program.

|Args|Description|
|-|-|
|-w|defines the wordlist to compare values to|
\-c|includes the ciphertext to be processed.|


---

###### [Home](https://mksipe.github.io/mksipe/)