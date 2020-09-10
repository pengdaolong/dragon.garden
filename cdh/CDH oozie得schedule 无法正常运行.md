## 问题描述
### oozie error 'NoneType' object has no attribute 'is_superus… 
## 问题解决
1. 修改oozie源代码
`/opt/cloudera/parcels/CDH-6.0.1-1.cdh6.0.1.p0.590678/lib/hue/apps/oozie/src/oozie/models2.py`
![file](/upload/2019/1/image-155114870784520190226103827685.png)
```
      + wf = Workflow(data=wf_doc.data,user=self.document.owner)
```
```
      return Workflow(document=wf_doc,user=self.document.owner)
```
