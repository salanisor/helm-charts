### helm-charts
Helm chart repository

##### Override Argo CD Health Check
As I mentioned before, we need to override the default Argo CD health check for the Subscription CR. Normally, Argo CD just creates the Subscription objects and doesn’t wait until the operator is installed on the cluster. In order to do that, we need to verify the value of the `status.state` field. If it equals the `AtLatestKnown` value, it means that the operator has been successfully installed. In that case, we can set the value of the Argo CD health check to Healthy. We can also override the default health check description to display the current version of the operator (the `status.currentCSV` field). If you installed Argo CD using Helm chart you can provide your health check implementation directly in the argocd-cm ConfigMap.

For those of you, who installed Argo CD using the operator (including me) there is another way to override the health check. We need to provide it inside the `extraConfig` field in the `ArgoCD` CR.

~~~
apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: openshift-gitops
  namespace: openshift-gitops
spec:
  ...
  extraConfig:
    resource.customizations: |
      operators.coreos.com/Subscription:
        health.lua: |
          hs = {}
          hs.status = "Progressing"
          hs.message = ""
          if obj.status ~= nil then
            if obj.status.state ~= nil then
              if obj.status.state == "AtLatestKnown" then
                hs.message = obj.status.state .. " - " .. obj.status.currentCSV
                hs.status = "Healthy"
              end
            end
          end
          return hs
~~~
