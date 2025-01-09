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
                age:
                  type: integer
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
  age: 30
  email: operator-sample@stable.example.com
```

</p>
</details>

### Listing operator

<details><summary>show</summary>
<p>

Use singular, plural and short forms

```bash
k get operator
k get operators
k get op
```

</p>
</details>
