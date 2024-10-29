# Use the official Ubuntu 22.04 LTS image
FROM ubuntu:22.04

# Set environment variables for non-interactive installs and locale
ENV DEBIAN_FRONTEND=noninteractive LANG=C.UTF-8 LC_ALL=C.UTF-8

# Install required packages
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y \
    wget \
    curl \
    python3 \
    python3-pip \
    openssh-server \
    sudo && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Create a non-root user 'hari' with a home directory
RUN useradd -m -d /home/hari -s /bin/bash hari && \
    echo "hari ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Set the custom hostname and configure PS1 with color
ENV CUSTOM_HOSTNAME=ubuntu-server
RUN echo "export PS1='\[\e[1;32m\]\u@$CUSTOM_HOSTNAME:\w\[\e[0m\]\\$ '" >> /home/hari/.bashrc

# Switch to user 'hari' for miniconda installation
USER hari
WORKDIR /home/hari

# Download and install Miniconda for user 'hari'
RUN wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O /home/hari/miniconda.sh && \
    bash /home/hari/miniconda.sh -b -p /home/hari/miniconda && \
    rm /home/hari/miniconda.sh

# Set conda path and initialize conda
ENV PATH="/home/hari/miniconda/bin:$PATH"
RUN echo ". /home/hari/miniconda/etc/profile.d/conda.sh" >> /home/hari/.bashrc && \
    conda init bash

# Install additional Python packages (example with numpy and pandas)
RUN conda install -y numpy pandas && \
    conda clean -a -y

# Switch back to root to clean up
USER root

# Remove unwanted cache and other temp files
RUN apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Expose the SSH port (optional)
EXPOSE 22

# Set the default user and working directory
USER hari
WORKDIR /home/hari

# Default command to run when starting the container
CMD ["/bin/bash"]
