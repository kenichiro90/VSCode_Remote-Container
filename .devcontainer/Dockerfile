# Pull image from ubuntu
FROM ubuntu:latest

# Add User
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Configure apt and install packages
RUN apt update
RUN apt install -y python3 python3-pip python3-venv iputils-ping less mysql-client net-tools vim
RUN pip3 install --no-cache-dir --upgrade pip
RUN pip3 install --no-cache-dir pylint jupyter matplotlib numpy pandas scipy django mysql-connector-python-rf

# Create a non-root user to use
RUN groupadd --gid $USER_GID $USERNAME
RUN useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME

# Add and support sudo for non-root user
RUN apt install -y sudo
RUN echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME
RUN chmod 0440 /etc/sudoers.d/$USERNAME

# Clean up
RUN apt autoremove -y
RUN apt clean -y 
RUN rm -rf /var/lib/apt/lists/*

# Open ports
EXPOSE 3306
EXPOSE 8000
EXPOSE 8888