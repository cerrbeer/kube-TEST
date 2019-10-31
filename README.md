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
