
## Checking Network Policies
To check the existing network policies in the `kafka` namespace, run:
```sh
kubectl get networkpolicy -n kafka
```
Expected output:
```
NAME            POD-SELECTOR                 AGE
kafka-network   network/kafka-network=true   XXm
```
This indicates that a network policy (`kafka-network`) is applied in the `kafka` namespace, which could restrict network traffic.

## Troubleshooting Network Policy Restrictions

### Step 1: Inspect the Network Policy
Check the specifics of the network policy to determine allowed and restricted traffic:
```sh
kubectl describe networkpolicy kafka-network -n kafka
```
Example output:
```
Name:         kafka-network
Namespace:    kafka
Created on:   2025-02-12 10:39:23 +0100 CET
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     network/kafka-network=true
  Allowing ingress traffic:
    To Port: <any> (traffic allowed to all ports)
    From:
      PodSelector: network/kafka-network=true
  Not affecting egress traffic
  Policy Types: Ingress
```

This policy only allows ingress traffic from pods with the label `network/kafka-network=true`.

### Step 2: Resolving Access Issues
If your `kcat` pod is unable to connect to Kafka, it is likely due to network policy restrictions. There are two possible solutions:

#### **Option 1: Add the Required Label to kcat Pod**
To allow your `kcat` pod to communicate with Kafka under the current policy, add the required label:
```sh
kubectl label pod [kcat-pod-name] network/kafka-network=true -n kafka
```
This ensures that the `kcat` pod matches the allowed selector in the network policy.

#### **Option 2: Modify the Network Policy to Allow All Pods**
If you want to permit communication from all pods in the namespace, update the network policy as follows:

Create a file `kafka-network-policy.yaml` with the following content:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: kafka-network
  namespace: kafka
spec:
  podSelector:
    matchLabels:
      network/kafka-network: "true"
  ingress:
  - from:
    - podSelector: {}  # Allow all pods in namespace
  egress:
  - to:
    - podSelector: {}
  policyTypes:
  - Ingress
```
Apply the updated policy:
```sh
kubectl apply -f kafka-network-policy.yaml
```

### Step 3: Test Kafka Connectivity
Once the policy is updated, test connectivity using `kcat`:
```sh
echo "Test Message" | kcat -P -b kafka:29092 -t testtopic
```
If the message is successfully sent, the network policy change has resolved the issue.


