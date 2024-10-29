# test
# Base image
FROM ubuntu:22.04-slim

# Environment variables
ENV DEBIAN_FRONTEND=noninteractive LANG=C.UTF-8 LC_ALL=C.UTF-8

# Install required packages and dependencies
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y wget curl python3 python3-pip openssh-server sudo && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Create a non-root user 'hari' with home directory and sudo privileges
RUN useradd -m -d /home/hari -s /bin/bash hari && \
    echo "hari ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Switch to user 'hari'
USER hari
WORKDIR /home/hari

# Download and install a specific version of Miniconda for user 'hari'
RUN wget https://repo.anaconda.com/miniconda/Miniconda3-py39_4.10.3-Linux-x86_64.sh -O /home/hari/miniconda.sh && \
    bash /home/hari/miniconda.sh -b -p /home/hari/miniconda && \
    rm /home/hari/miniconda.sh

# Set up Conda in the user's .bashrc
ENV PATH="/home/hari/miniconda/bin:$PATH"
RUN echo ". /home/hari/miniconda/etc/profile.d/conda.sh" >> /home/hari/.bashrc && \
    conda init bash && \
    conda install -y numpy pandas && \
    conda clean -a -y

# Expose SSH port
EXPOSE 22

# Healthcheck for SSH (Optional)
HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:22 || exit 1

# Default shell
ENTRYPOINT ["/bin/bash"]
CMD ["-l"]
