# Admission Controllers
* Que es?
* Tipo
* Demo

Hace unos años, la mejor manera de administrar la seguridad dentro de Kubernetes era mediante RBAC sin embargo no proporciona la granularidad que en algunos casos se necesitaba. 

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: read-only
  namespace: default
rules:
- apiGroups:
  - ""
  resources: ["*"]
  verbs:
  - get
  - list
  - watch
```

Se diseñaro las Pod Security Policies (PSP) como posible solución, pero se tomó la decisión de deprecarlas en la vesión v1.21 por su complejidad de compresión y uso.

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: privileged
spec:
  fsGroup:
    rule: RunAsAny
  privileged: true
  runAsUser:
    rule: RunAsAny
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  volumes:
  - '*'
  allowedCapabilities:
  - '*'
  hostPID: true
  hostIPC: true
  hostNetwork: true
  hostPorts:
  - min: 1
    max: 65536
```

Por ello existe un gran debate entre [__Kyverno__](https://www.openpolicyagent.org/docs/v0.12.2/kubernetes-admission-control/) y [__OPA__](https://github.com/kyverno/kyverno), ambos son __Admissions Controllers__.

## OPA
* Permite la validación y mutación de recursos
* Pensado para sistemas fuera de Kubernetes
* [Gatekeeper](https://github.com/open-policy-agent/gatekeeper) es el webhook creado para el uso de OPA en Kubernetes 
* Casos de uso:
  * Cloudformation
  * HTTP APIs
  * Docker
  * Kafka
  * SSH y sudo
  * Terraform
* Las políticas se escriben en el lenguaje declarativo [REGO](https://www.openpolicyagent.org/docs/latest/policy-language/)
  * Utilidad para creación de políticas [REGO Playground](https://play.openpolicyagent.org/)


## Kyverno
* Específico para Kubernetes
* 
El más joven de los dos motores de gestión policial, Kyverno, ha sido desarrollado como una respuesta a la naturaleza altamente compleja y demasiado exigente de OPA Gatekeeper. Originalmente desarrollado por Nirmata y ahora parte de los proyectos sandbox de CNCF, este motor de políticas de Kubernetes brinda a los departamentos de DevOps una gran cantidad de beneficios, desde su diseño nativo de Kubernetes hasta su capacidad para generar políticas.

En respuesta a la compleja tarea de aprender el lenguaje Rego de la OPA, Kyverno está diseñado específicamente para usar una composición al estilo de Kubernetes. Esto significa que sus capacidades de expresión de políticas brindan a los usuarios un mayor grado de simplicidad y flexibilidad al tiempo que permiten el desarrollo de políticas más poderosas. Este diseño nativo de Kubernetes también hace que Kyverno sea altamente compatible con la configuración dinámica, GitOps y las herramientas que se aplican comúnmente en un entorno de Kubernetes.

Los motores de políticas se crean con capacidades clave que les permiten validar y modificar políticas según sea necesario. Kyverno, además de las funciones de validación y mutación que realiza OPA Gatekeeper, también tiene capacidades de generación integradas. Las reglas de generación permiten recursos complementarios y de apoyo cada vez que se actualiza o crea un recurso. 
