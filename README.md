# kube-TEST
Start Kubernetes cluster in VM + prometheus, run :  " vagrant up "

Prometheus:  http://192.168.205.11:30000/alerts 


Alerts:

      rules:
      
      
      - alert: KubeControllerManagerDown
        expr: count(container_last_seen{container_name="kube-controller-manager",instance="k8s-head"}) < 1
        for: 1m
        labels:
          job: kube-controller-manager
          severity: critical
        annotations:
          description: Prometheus can't scape metrics from kube-controller-manager(s).
          summary: Kube Controller Manager down
      
      
      
      - alert: NodeCpuHigh
        expr: (sum(rate(container_cpu_usage_seconds_total{id="/", container_name!="POD"}[1m])) by (kubernetes_io_hostname) / sum(machine_cpu_cores) by (kubernetes_io_hostname)  * 100)  > 85
        for: 15m
        labels:
          severity: warning
        annotations:
          summary: Node {{$labels.kubernetes_io_hostname}} has more than 85% allocatable cpu used.
          description: "Node CPU load is > 85%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels.kubernetes_io_hostname }}"   
     
     
     
     - alert: NonPodCpuHigh
        expr: sum (rate (process_cpu_seconds_total[15m])) / sum (machine_cpu_cores) * 100  > 35
        for: 15m
        labels:
          severity: warning
        annotations:
          summary: "High NonPod Processes CPU load (instance {{ $labels.instance }})"
          description: "High NonPod Processes CPU load is > 35%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}" 
      
      
      - alert: PodMemoryHigh
        expr: (sum(kube_pod_container_resource_requests_memory_bytes) by (namespace, pod, node) * on (pod) group_left()  (sum(kube_pod_status_phase{phase="Running"}) by (pod, namespace) == 1) / sum(kube_pod_container_resource_limits_memory_bytes) by (namespace, pod, node) * on (pod) group_left()  (sum(kube_pod_status_phase{phase="Running"}) by (pod, namespace) == 1)) * 100 > 70
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Pod named {{$labels.pod}} in {{$labels.namespace}} is using more than 70% of CPU Limit"
          description: "Pod Memory usage is above 70%\n  VALUE = {{ $value }}\n  LABELS: {{$labels.pod}} in {{$labels.namespace}}"    
