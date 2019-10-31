# kube-TEST
Start Kubernetes cluster in VM + prometheus, run :  " vagrant up "

Prometheus:  http://192.168.205.11:30000/alerts 


Alerts:

      rules:
      
      
     - alert: KubeControllerManagerDown
        expr: kube_pod_container_status_running{pod="kube-controller-manager-k8s-head"} == 0
        for: 1m
        labels:
          job: kube-controller-manager
          severity: critical
        annotations:
          description: Prometheus can't scape metrics from kube-controller-manager(s).
          summary: Kube Controller Manager down
      
      
      - alert: NodeCpuHigh
        expr: sum(rate(node_cpu_seconds_total{mode!="idle"}[15m])) without (mode,cpu) * 100   > 85
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: Node {{$labels.instance}} has more than 85% allocatable cpu used.
          description: "Node CPU load is > 85%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels.instance }}"   
      
      
      - alert: NonPodCpuHigh
        expr: sum(rate(container_cpu_usage_seconds_total{id="/", container_name!="POD"}[15m])) by (kubernetes_io_hostname) * 100 > 35
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "High NonPod Processes CPU load (instance {{ $labels.instance }})"
          description: "High NonPod Processes CPU load is > 35%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}" 
     
     
     - alert: PodMemoryHigh
        expr: sum(kube_pod_container_resource_requests_memory_bytes) by (node) / sum(kube_node_status_allocatable_memory_bytes) by (node) * 100 > 70
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "Pod named {{$labels.pod}} in {{$labels.node}} is using more than 70% of CPU Limit"
          description: "Pod Memory usage is above 70%\n  VALUE = {{ $value }}\n  LABELS: {{$labels.pod}} in {{$labels.node}}"    



Test Cases:

1. Stop container kube-controller-manager-k8s-head  

2,3. Run  "sudo stress --cpu 8 -v --timeout 900s"  on k8s-node-1

4. Run: 

kubectl run resource-consumer --image=gcr.io/kubernetes-e2e-test-images/resource-consumer:1.4 --expose --service-overrides='{ "spec": { "type": "LoadBalancer" } }' --port 8181 --requests='cpu=500m,memory=256Mi'


curl --data "millicores=300&durationSec=600" http://<EXTERNAL-IP>:8181/ConsumeCPU
