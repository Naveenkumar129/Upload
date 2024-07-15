---
- name: Kafka cluster region switch playbook
  hosts: all
  become: yes
  tasks:
    - name: Switch to West Cluster - Step 1
      shell: sudo sh /apps/confluent/scripts/dr/get_leader_count_per_broker.sh
      register: switch_west_step1
      ignore_errors: yes
      tags: switch_west

    - name: Switch to West Cluster - Step 2
      shell: sudo sh /apps/confluent/scripts/dr/apply_unclean_election_2_noleader_topics.sh
      register: switch_west_step2
      ignore_errors: yes
      tags: switch_west

    - name: Switch to West Cluster - Step 3
      shell: sudo sh /apps/confluent/scripts/dr/apply_rpp_to_async_topics.sh async-topics-failover-2-us-west-2.json
      register: switch_west_step3
      ignore_errors: yes
      tags: switch_west

    - name: Switch to West Cluster - Step 4
      shell: sudo sh /apps/confluent/scripts/dr/apply_rpp_to_sync_topics.sh_sync-topics-failover-2-us-west-2.json
      register: switch_west_step4
      ignore_errors: yes
      tags: switch_west

    - name: Switch to West Cluster - Step 5
      shell: sudo sh /apps/confluent/scripts/dr/apply_auto_data_rebalancer_async_topics.sh
      register: switch_west_step5
      ignore_errors: yes
      tags: switch_west

    - name: Switch to West Cluster - Step 6
      shell: sudo sh /apps/confluent/scripts/dr/get_leader_count_per_broker.sh
      register: switch_west_step6
      ignore_errors: yes
      tags: switch_west

    - name: Switch to East Cluster - Step 1
      shell: sudo sh /apps/confluent/scripts/dr/get_leader_count_per_broker.sh
      register: switch_east_step1
      ignore_errors: yes
      tags: switch_east

    - name: Switch to East Cluster - Step 2
      shell: sudo sh /apps/confluent/scripts/dr/apply_unclean_election_2_noleader_topics.sh
      register: switch_east_step2
      ignore_errors: yes
      tags: switch_east

    - name: Switch to East Cluster - Step 3
      shell: sudo sh /apps/confluent/scripts/dr/apply_rpp_to_async_topics.sh async-topics-failover-2-us-east-1.json
      register: switch_east_step3
      ignore_errors: yes
      tags: switch_east

    - name: Switch to East Cluster - Step 4
      shell: sudo sh /apps/confluent/scripts/dr/apply_rpp_to_sync_topics.sh sync-topics-failover-2-us-east-1.json
      register: switch_east_step4
      ignore_errors: yes
      tags: switch_east

    - name: Switch to East Cluster - Step 5
      shell: sudo sh /apps/confluent/scripts/dr/apply_auto_data_rebalancer_async_topics.sh
      register: switch_east_step5
      ignore_errors: yes
      tags: switch_east

    - name: Switch to East Cluster - Step 6
      shell: sudo sh /apps/confluent/scripts/dr/get_leader_count_per_broker.sh
      register: switch_east_step6
      ignore_errors: yes
      tags: switch_east
---------

pipeline {
    agent any

    parameters {
        choice(name: 'ACTION', choices: ['switch_west', 'switch_east'], description: 'Choose the Kafka cluster switch action')
    }

    environment {
        INVENTORY = 'path/to/inventory'
        PLAYBOOK = 'path/to/kafka_cluster_switch_playbook.yml'
    }

    stages {
        stage('Run Ansible Playbook') {
            steps {
                script {
                    if (params.ACTION == 'switch_west') {
                        sh """
                        ansible-playbook -i ${INVENTORY} ${PLAYBOOK} --tags "switch_west"
                        """
                    } else if (params.ACTION == 'switch_east') {
                        sh """
                        ansible-playbook -i ${INVENTORY} ${PLAYBOOK} --tags "switch_east"
                        """
                    }
                }
            }
        }
    }
}

----

[kafka_clusters]
kafka-node-1 ansible_host=192.168.1.101 ansible_user=your_user
kafka-node-2 ansible_host=192.168.1.102 ansible_user=your_user
kafka-node-3 ansible_host=192.168.1.103 ansible_user=your_user

[all:vars]
ansible_python_interpreter=/usr/bin/python3





ker 4- partition on broker 4
ish-4.25 ./kafka-producer-perf-test.sh -.throughput 2000 --record-size 80000 --topic petros-test-1p-0rf-b --producer.config /config/producer.properties --num-records 2400
props batch.size=524288 linger.ms=20
2022-10-10 15:20:50,024] INFO ProducerConfig values:


**6. Troubleshooting Kubernetes Networking**
Common Issues and Solutions

Pods can't communicate: Check CNI plugin status and network policies.

Service not reachable: Verify Service and Endpoint configurations.

DNS resolution issues: Ensure CoreDNS is running and configured correctly.


1. Pods Can't Communicate
Check CNI Plugin Status

The Container Network Interface (CNI) plugin is responsible for networking in your Kubernetes cluster. Issues with the CNI plugin can disrupt pod-to-pod communication.

Check CNI plugin pods:

bash
Copy code
kubectl get pods -n kube-system -l k8s-app=flannel
kubectl get pods -n kube-system -l k8s-app=calico-node
Adjust the label according to the CNI plugin used (e.g., flannel, calico, weave, etc.).

Check logs for CNI plugin pods:

bash
Copy code
kubectl logs -n kube-system <cni-pod-name>
Check CNI configuration files (usually in /etc/cni/net.d/):

bash
Copy code
ls /etc/cni/net.d/
cat /etc/cni/net.d/<cni-config-file>
Check Network Policies

Network policies control the traffic between pods.

List all network policies:

bash
Copy code
kubectl get networkpolicies --all-namespaces
Describe network policies in a specific namespace:

bash
Copy code
kubectl describe networkpolicy -n <namespace>
Example of allowing all traffic (for debugging):

yaml
Copy code
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all
  namespace: <namespace>
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - {}
  egress:
  - {}
2. Service Not Reachable
Verify Service Configuration

Get service details:

bash
Copy code
kubectl get svc -n <namespace>
kubectl describe svc <service-name> -n <namespace>
Check service endpoints:

bash
Copy code
kubectl get endpoints -n <namespace>
kubectl describe endpoints <service-name> -n <namespace>
Common issues to check:

Ensure the service type is correct (ClusterIP, NodePort, LoadBalancer).
Verify that the selector labels match the labels of the target pods.
Check if the service port and target port are correctly defined.
3. DNS Resolution Issues
Ensure CoreDNS is Running

CoreDNS provides DNS resolution for the Kubernetes cluster.

Check CoreDNS pods:

bash
Copy code
kubectl get pods -n kube-system -l k8s-app=kube-dns
Check logs for CoreDNS pods:

bash
Copy code
kubectl logs -n kube-system <coredns-pod-name>
Check CoreDNS ConfigMap

Get CoreDNS ConfigMap:
bash
Copy code
kubectl get configmap coredns -n kube-system -o yaml
Test DNS Resolution

Create a busybox pod for testing:

bash
Copy code
kubectl run -i --tty --rm dnsutils --image=gcr.io/kubernetes-e2e-test-images/dnsutils:1.3 --restart=Never -- bash
Inside the busybox pod, test DNS resolution:

bash
Copy code
nslookup kubernetes.default
nslookup <service-name>.<namespace>.svc.cluster.local
Example Commands for Troubleshooting
Check Pod-to-Pod Communication

Ping between pods (using a test pod):
bash
Copy code
kubectl run test-pod --rm -i --tty --image=busybox -- sh
ping <target-pod-ip>
Check Service Reachability

Curl service endpoint (from a pod):
bash
Copy code
kubectl run curl-pod --rm -i --tty --image=radial/busyboxplus:curl -- sh
curl <service-ip>:<port>
Summary
Pods Can't Communicate: Ensure the CNI plugin is running and configured correctly, and verify network policies.
Service Not Reachable: Check service and endpoint configurations, ensure the correct ports and selectors.
DNS Resolution Issues: Confirm CoreDNS is running and properly configured, and test DNS resolution using a test pod.
By following these steps and using the provided commands, you should be able to diagnose and resolve common networking issues in your Kubernetes cluster.





