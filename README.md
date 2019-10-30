# kube-TEST
Start Kubernetes cluster in VM + prometheus, run :  " vagrant up "; 

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
        expr: sum (rate (container_cpu_usage_seconds_total{id="/"}[15m])) / sum (machine_cpu_cores) * 100  > 85
        for: 15m
        labels:
          severity: warning
        annotations:
          summary: "High Node CPU load (instance {{ $labels.instance }})"
          description: "Node CPU load is > 85%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"   
      
      - alert: NonPodCpuHigh
        expr: sum (rate (process_cpu_seconds_total[15m])) / sum (machine_cpu_cores) * 100  > 35
        for: 15m
        labels:
          severity: warning
        annotations:
          summary: "High NonPod Processes CPU load (instance {{ $labels.instance }})"
          description: "High NonPod Processes CPU load is > 35%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}" 
      
      - alert: PodMemoryHigh
        expr: (sum(container_memory_usage_bytes) BY (ip) / sum(container_memory_max_usage_bytes) BY (ip) * 100) > 70
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Pod Memory usage (instance)"
          description: "Pod Memory usage is above 70%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"    
