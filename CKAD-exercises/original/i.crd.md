# Extend the Kubernetes API with CRD (CustomResourceDefinition)

- Note: CRD is part of the new CKAD syllabus. Here are a few examples of installing custom resource into the Kubernetes API by creating a CRD.

## CRD in K8s

### Create a CustomResourceDefinition manifest file for an Operator with the following specifications :

- _Name_ : `operators.stable.example.com`
- _Group_ : `stable.example.com`
- _Schema_: `<email: string><name: string><age: integer>`
- _Scope_: `Namespaced`
- _Names_: `<plural: operators><singular: operator><shortNames: op>`
- _Kind_: `Operator`

<details><summary>show</summary>
<p>

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: operators.stable.example.com
spec:
  group: stable.example.com
  scope: Namespaced
  names:
    plural: operators
    singular: operator
    kind: Operator
    shortNames:
      - op
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                email:
                  type: string
                name:
                  type: string
                age:
                  type: string
```

```bash
 k create -f crd.yaml
customresourcedefinition.apiextensions.k8s.io/operators.stable.example.com created

k api-resources | grep operators
operators                           op           stable.example.com/v1             true         Operator
```

</p>
</details>

### Create the CRD resource in the K8S API

<details><summary>show</summary>
<p>

</p>
</details>

### Create custom object from the CRD

- _Name_ : `operator-sample`
- _Kind_: `Operator`
- Spec:
  - email: `operator-sample@stable.example.com`
  - name: `operator sample`
  - age: `30`

<details><summary>show</summary>
<p>

```yaml
apiVersion: stable.example.com/v1
metadata:
  name: operator-sample
kind: Operator
spec:
  email: "operator-sample@stable.example.com"
  name: "operator sample"
  age: 30
```

</p>
</details>

### Listing operator

<details><summary>show</summary>
<p>

Use singular, plural and short forms

```bash
controlplane ~ ➜  k get op
NAME              AGE
operator-sample   3s

controlplane ~ ➜  k get operator
NAME              AGE
operator-sample   8s

controlplane ~ ➜  k get operators
NAME              AGE
operator-sample   10s
```

</p>
</details>
