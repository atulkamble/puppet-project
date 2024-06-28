## Project on Puppet for DevOps on Amazon EC2: manage configurations and automate deployments.

Here's a step-by-step guide to create a project:

### Project Overview

The goal of this project is to set up a Puppet master and a Puppet agent on Amazon EC2 instances. Puppet will be used to automate the configuration of the agent instance.

### Prerequisites

1. **AWS Account**: Ensure you have an AWS account set up.
2. **IAM Role**: Create an IAM role with the necessary permissions to manage EC2 instances.
3. **Key Pair**: Create an EC2 key pair for SSH access to the instances.
4. **Security Groups**: Create security groups to allow SSH access (port 22) and Puppet communication (port 8140).

### Step 1: Launch EC2 Instances

1. **Puppet Master Instance**:
    - Launch an Amazon Linux 2 instance.
    - Configure the instance with the appropriate IAM role and security group.
    - SSH into the instance and install Puppet server.

2. **Puppet Agent Instance**:
    - Launch another Amazon Linux 2 instance.
    - Configure it similarly with the appropriate IAM role and security group.
    - SSH into the instance and install Puppet agent.

### Step 2: Install and Configure Puppet

1. **On Puppet Master Instance**:
    - Update the package repository and install Puppet server:
      ```sh
      sudo yum update -y
      sudo amazon-linux-extras install epel -y
      sudo rpm -Uvh https://yum.puppet.com/puppet7-release-el-7.noarch.rpm
      sudo yum install puppetserver -y
      ```

    - Start and enable the Puppet server:
      ```sh
      sudo systemctl start puppetserver
      sudo systemctl enable puppetserver
      ```

    - Configure Puppet server memory settings in `/etc/sysconfig/puppetserver`:
      ```sh
      sudo sed -i 's/JAVA_ARGS.*/JAVA_ARGS="-Xms512m -Xmx512m -XX:MaxPermSize=256m -XX:ReservedCodeCacheSize=256m"/' /etc/sysconfig/puppetserver
      sudo systemctl restart puppetserver
      ```

2. **On Puppet Agent Instance**:
    - Update the package repository and install Puppet agent:
      ```sh
      sudo yum update -y
      sudo amazon-linux-extras install epel -y
      sudo rpm -Uvh https://yum.puppet.com/puppet7-release-el-7.noarch.rpm
      sudo yum install puppet-agent -y
      ```

    - Start and enable the Puppet agent:
      ```sh
      sudo systemctl start puppet
      sudo systemctl enable puppet
      ```

    - Configure the Puppet agent to connect to the Puppet master:
      ```sh
      sudo /opt/puppetlabs/bin/puppet resource service puppet ensure=running enable=true
      sudo /opt/puppetlabs/bin/puppet agent --server=<PUPPET_MASTER_PRIVATE_IP> --test
      ```

### Step 3: Sign the Agent Certificate on the Master

1. **On Puppet Master Instance**:
    - List the pending certificate requests:
      ```sh
      sudo /opt/puppetlabs/bin/puppetserver ca list
      ```

    - Sign the agent certificate:
      ```sh
      sudo /opt/puppetlabs/bin/puppetserver ca sign --certname <AGENT_CERTNAME>
      ```

### Step 4: Create a Puppet Manifest

1. **On Puppet Master Instance**:
    - Create a simple manifest to manage the state of a resource, such as creating a file on the agent:
      ```sh
      sudo mkdir -p /etc/puppetlabs/code/environments/production/manifests
      sudo nano /etc/puppetlabs/code/environments/production/manifests/site.pp
      ```

    - Add the following content to `site.pp`:
      ```puppet
      node default {
        file { '/tmp/puppet_test_file':
          ensure  => 'present',
          content => 'This file is managed by Puppet.',
        }
      }
      ```

### Step 5: Test the Configuration

1. **On Puppet Agent Instance**:
    - Trigger a Puppet agent run to apply the manifest:
      ```sh
      sudo /opt/puppetlabs/bin/puppet agent --test
      ```

2. **Verify**:
    - Check the `/tmp` directory for the `puppet_test_file` to ensure it has been created and managed by Puppet.

### Documentation

1. **Project Overview Document**: Summarize the project goals, prerequisites, and a high-level description of the steps.
2. **Installation Guide**: Detailed step-by-step guide for installing and configuring Puppet on both master and agent instances.
3. **Manifest Documentation**: Explanation of the Puppet manifest, including the purpose of each configuration.
4. **Testing and Validation**: Instructions on how to test and validate the setup.

### Conclusion

This project provides a basic introduction to using Puppet for configuration management on Amazon EC2 instances. It covers the essentials of setting up a Puppet master and agent, creating and applying a simple manifest, and documenting the process. This setup can be expanded with more complex configurations and additional Puppet features as needed.
