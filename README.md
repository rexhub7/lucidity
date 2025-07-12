# CloudWatch install using Ansible

**Prerequisites**
* Ansible: Ensure Ansible is installed on management/control instance.
  * Use `ansible.cfg` to add the plugins and inventory details.
* AWS CLI: The AWS CLI should be installed and configured with appropriate IAM permission on the target instances. 
* IAM Role: The EC2 instances need an IAM role with the `CloudWatchAgentServerPolicy` attached to allow them to interact with CloudWatch. 
* CloudWatch Agent Configuration: You'll need a `file_config.json` file that defines what metrics the agent should collect. 
* Target Instances: Identify the Amazon Linux instances you want to manage.
  * Filter the instances in `aws_ec2.yaml`

**Ansible Playbook**

Create and apply the ansible playbook to install Cloudwatch agent in EC2 instances.

```yaml
- name: Install CloudWatch Agent
  hosts: all    #This targets all hosts defined in Ansible inventory.
  become: yes    #This ensures the tasks are executed with elevated privileges.

  tasks:
    - name: Download CloudWatch Agent installer
      get_url:        # Downloads the CloudWatch agent package from the specified URL.
        url: https://amazoncloudwatch-agent.s3.amazonaws.com/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm #https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
        dest: ./amazon-cloudwatch-agent.rpm
        mode: '0644'

    - name: Install CloudWatch Agent
      shell: sudo rpm -U ./amazon-cloudwatch-agent.rpm 

    - name: Copy config file
      template:
        src: /root/file_config.json
        dest: /home/file_config.json

    - name: Moving config
      shell: sudo mv /home/file_config.json /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.d/file_config.json

    - name: Starting CloudWatch
      shell: sudo systemctl start  amazon-cloudwatch-agent

    - name: Automatic Start CloudWatch
      shell: sudo systemctl enable  amazon-cloudwatch-agent      

    - name: Check CloudWatch agent status
      command: sudo systemctl status  amazon-cloudwatch-agent
      register: cw_agent_status

    - debug:
        var: cw_agent_status.stdout_lines
```

**Validation**

Make sure the necessary metrics are available in CloudWatch.
