FROM quay.io/dovid/ansible-ee-azure:1.1

USER root
RUN rpm --import https://packages.microsoft.com/keys/microsoft.asc && rpm -Uvh https://packages.microsoft.com/config/rhel/8/packages-microsoft-prod.rpm 
RUN microdnf install -y azure-cli

USER 1000
ENTRYPOINT ["/opt/builder/bin/entrypoint", "dumb-init"]
CMD ["bash"]
