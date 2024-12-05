# Autoscaling on Openstack
  ## Download Template and Binary
    git clone https://github.com/teghitsugaya/autoscaling-osp.git
    cd autoscaling-osp
    chmod +x autoscalingctl
    cp autoscalingctl /usr/local/bin/

  ## Test Query Binary
    source ~/<your keystone.rc>
    autoscalingctl list

  ## Create Autoscaling
  #### Edit the instance.yaml file and change based your requierement
    parameters:
      image:
        default: "Ubuntu Server 22.04"
      flavor:
        default: "GP.1C1G"
      key_name:
        default: "remote-server"
      network:
        default: "Subnet_Private01_DC1"
      availability_zone:
        default: "AZ_Private01_DC1"
    
    port:
        security_groups:
          - allow-all
          
    volume:
        size: 10

  #### Edit the template.yaml file and change based your requierement
  #### choose the threshold alarm_high (50% - 80%) / instance will do scale out if the resource cpu have more than the threshold
    cpu_alarm_high:
      type: OS::Aodh::GnocchiAggregationByResourcesAlarm
      properties:
        description: Scale up if CPU > 60%
        metric: cpu
        aggregation_method: rate:mean
        granularity: 300
        evaluation_periods: 1 
          #threshold: 240000000000.0   #80%
          #threshold: 210000000000.0   #70%
        threshold: 180000000000.0   #60%
          #threshold: 150000000000.0   #50%
          
   #### choose the threshold alarm_low (50% - 80%) / instance will do scale down if the resource cpu have less than the threshold
    cpu_alarm_low:
      type: OS::Aodh::GnocchiAggregationByResourcesAlarm
      properties:
        description: Scale down if CPU > 10%
        metric: cpu
        aggregation_method: rate:mean
        granularity: 300
        evaluation_periods: 3
          #threshold: 45000000000.0  #15%
        threshold: 30000000000.0  #10%
          #threshold: 15000000000.0  #5%

   ## Create the autoscaling
      autoscaling create <name>
      autoscaling create myinstance
      
   ## Show list resource autoscaling
      autoscaling list
      
   ## Show status resource autoscaling
      autoscaling show status <name>
      autoscaling show status myinstance-xxxx

   ## Watch the CPU Usage Autoscaling resources (the metrics utilyze cpu usage will be appear in every 300s)
      watch autoscaling show cpu-usage <name>
      watch autoscaling show cpu-usage myinstance-xxxx 
        
   ## Genereate utilize CPU the instances
      ssh ubuntu@<ipaddress>
      sudo su
      apt update
      apt install stress-ng
      stress-ng --cpu 4
      htop
      
   ### Back to CPU Usage Autoscaling resources tab, and look the cpu usages, its will be increases (the metrics utilyze cpu usage will be appear in every 300s)
   ## Back to the Status resources autoscaling, observe changes in alarm status, alarm status high will be change to alert , and amount of instances
      watch autoscaling show status <name>
      watch autoscaling show status myinstance-xxxx
