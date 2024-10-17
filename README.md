# Ansible variables

## Motivation

when working with `Ansible` you will at some point in time want to use variables to avoid repetition.

Note: `Ansible` needs SSH access to the target machine. You can find out how to configure SSH access in the docker container at [https://github.com/Frunza/configure-docker-container-with-ssh-access](https://github.com/Frunza/configure-docker-container-with-ssh-access)

## Prerequisites

A Linux or MacOS machine for local development. If you are running Windows, you first need to set up the *Windows Subsystem for Linux (WSL)* environment.

You need `docker cli` and `docker-compose` on your machine for testing purposes, and/or on the machines that run your pipeline.
You can check both of these by running the following commands:
```sh
docker --version
docker-compose --version
```

Make sure that you already have a docker container with SSH access.

## Implementation

If there are more places in your playlist where you want to use the same value in more tasks, you can declare variables as shown below:
```sh
  vars:
    myLocalVariable: "value of my local variable"
    myLocalVariable2: "value of my local variable2"
```
You can use these variables in various tasks as shown below:
```sh
    - name: Print value of myLocalVariable
      debug:
        msg: "myLocalVariable: {{ myLocalVariable }}"

    - name: Print value of myLocalVariable2
      debug:
        msg: "myLocalVariable2: {{ myLocalVariable2 }}"
```

While this works fine for your playlist, you might run into cases where you want to use the same variables in more than one playlist. To achieve this you can create a file to store your variables and import it in the playlist you want to use the defined variables.

To put your variables into a file, create a new file named *myVars.yml* and fill it with
```sh
appBaseDir: /data/myApp
```

To use this variable into your playlist, you must first import the *myVars.yml* file
```sh
  vars_files:
    - myVars.yml
```
Afterwards you can just use the variables as shown before.

When using path variables you could run into duplication cases. Take for example
```sh
appBaseDir: /data/myApp
appDataDir: /data/myApp/data
```
You probably notice the */data/myApp* duplication. To address this, you can improve your variables as shown here:
```sh
appBaseDir: /data/myApp
appDataDir: "{{ appBaseDir }}/data"
```

## Usage

Navigate to the root of the repository and run the following command:
```sh
sh run.sh 
```

The following happens:
1) the first command builds the docker image, passing the private key value as an argument and tagging it as *variablesansible*
2) the docker image sets up the SSH access by copying the value of the `SSH_PRIVATE_KEY` argument to the standard location for SSH keys
3) the second command uses docker-compose to run the `Ansible` playbook that prints out the values of various variables

Note: if you want to test this locally, consider changing the `hosts` in the `Ansible` playbook to `local`.
