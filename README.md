KYVERNO - Very simple policies and exception 

kyverno-cli
> kyverno version
Version: 1.11.2
Time: ---
Git commit ID: ---


POD WITH CERTAIN LABELS REQUIRE:
ClusterPolicy
label = deny

apply pod - it should be created
Delete newly created pod
apply policy: Require-Labels
Verify the policy is created
Describe the policy
apply pod - should not be able to be created
Fix the pod with the error message by kyverno
What is the difference for Audit Vs EnForce
and how can you verify that Audit is working and read the output

POD within excluded namespace is not affected by above cluster policy:
this pod yaml definition is now denied across the whole cluster within all namespaces
Create namespace policy for allow this pod to be created within a specific namespace
Understand the WARN
Modify the  pod yaml file so it is affected by the exception newly created and try to create the pod now
remove the namespace values and try to create it


NOTE:
Don't forget to check that the kyverno has enablePolicyException enabled
https://www.youtube.com/watch?v=Evx1frnQMhw&t=595s
This is the admission-controller deployment that sets this during the creation of the container spec
> kubectl get deploy -n kyverno kyverno-admission-controller -o yaml | grep "Exception"




Kyverno COMMANDS
Verify creation:
> kubectl get clusterpolicies.kyverno.io -A
Verify the kyverno events
> kubectl describe clusterpolicies.kyverno.io require-labels

kyverno-deny-pods-cluster-wide-with-label-kubernetes-io-deny.yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-labels
  annotations:
    policies.kyverno.io/title: Require Labels
    policies.kyverno.io/category: Best Practices
    policies.kyverno.io/minversion: 1.6.0
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod, Label
    policies.kyverno.io/description: >-
      Define and use labels that identify semantic attributes of your application or Deployment.
      A common set of labels allows tools to work collaboratively, describing objects in a common manner that
      all tools can understand. The recommended labels describe applications in a way that can be
      queried. This policy validates that the label `app.kubernetes.io/name` is specified with some value.      
spec:
  validationFailureAction: Enforce
  background: true
  rules:
  - name: check-for-labels
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "The label `app.kubernetes.io/name` is required."
      pattern:
        metadata:
          labels:
            app.kubernetes.io/name: "?*"



pod-deny-label.yaml

apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  labels:
    deny: "true"
spec:
  containers:
  - name: nginx
    image: nginx:latest




kyverno-PolicyException-NS.yaml
apiVersion: kyverno.io/v2alpha1
kind: PolicyException
metadata:
  name: nginx-ns-exception
  namespace: nginx-ns
spec:
  exceptions:
  - policyName: require-labels
    ruleNames:
    - check-for-labels
  match:
    any:
    - resources:
        kinds:
        - Pod
        namespaces:
        - nginx-ns
        names:
        - test-deny-*







