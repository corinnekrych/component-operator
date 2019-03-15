# component-operator

Cluster-scoped test operator generated following [instructions](https://github.com/operator-framework/operator-sdk/blob/master/doc/user-guide.md) 

* generate code
```
operator-sdk new component-operator --cluster-scoped=true
cd component-operator
operator-sdk add api --api-version=devopsconsole.corinnekrych/v1alpha1 --kind=Component
operator-sdk add controller --api-version=devopsconsole.corinnekrych/v1alpha1 --kind=Component
operator-sdk generate k8s
```
* modify component controller, add logs:
```
	log.Info("============================================================")
	log.Info(fmt.Sprintf("***** Reconciling Component %s, namespace %s", request.Name, request.Namespace))
	log.Info(fmt.Sprintf("** Creation time : %s", instance.ObjectMeta.CreationTimestamp))
	log.Info(fmt.Sprintf("** Resource version : %s", instance.ObjectMeta.ResourceVersion))
	log.Info(fmt.Sprintf("** Generation version : %s", strconv.FormatInt(instance.ObjectMeta.Generation, 10)))
	log.Info(fmt.Sprintf("** Deletion time : %s", instance.ObjectMeta.DeletionTimestamp))
	log.Info("============================================================")

```
* build, push image
```
minishift start
oc login -u system:admin
oc new-project devopsconsole
oc create -f deploy/crds/devopsconsole_v1alpha1_component_crd.yaml
operator-sdk build quay.io/corinnekrych/component-operator:v0.0.1
sed -i "" 's|REPLACE_IMAGE|quay.io/corinnekrych/component-operator:v0.0.1|g' deploy/operator.yaml
docker login -u corinnekrych quay.io
docker push quay.io/corinnekrych/component-operator:v0.0.1
```
> NOTE: Go to qay.io, make the image public

* deploy operator
```
export OPERATOR_NAMESPACE=$(kubectl config view --minify -o jsonpath='{.contexts[0].context.namespace}')
sed -i "" "s|REPLACE_NAMESPACE|$OPERATOR_NAMESPACE|g" deploy/role_binding.yaml
oc create -f deploy/service_account.yaml
oc create -f deploy/role.yaml
oc create -f deploy/role_binding.yaml
oc create -f deploy/operator.yaml
```
* test with CR in another ns
```
oc new-project another
oc apply -f deploy/crds/devopsconsole_v1alpha1_component_cr.yaml
oc get all,is,component,bc,build,deployment,pod
NAME                        READY     STATUS    RESTARTS   AGE
pod/example-component-pod   1/1       Running   0          27s

NAME                                                     AGE
component.devopsconsole.corinnekrych/example-component   27s
```
