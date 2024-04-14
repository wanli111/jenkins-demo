```
###下载地址

https://github.com/helm/helm/releases
wget https://get.helm.sh/helm-v3.12.3-linux-amd64.tar.gz
tar zxf helm-v3.12.3-linux-amd64.tar.gz
cd linux-amd64/ &&  mv helm /usr/local/bin/ &&  chmod +x /usr/local/bin/helm 
```

基本使用

```
### 添加一个仓库
helm repo add stable http://mirror.azure.cn/kubernetes/charts/
### 查看仓库
helm repo list
### 搜索可以安装的 chart 包
helm search repo stable
### 更新仓库，获取最新的 chart 列表
helm repo update
### 示例 --generate-name 生成随机的 release 名称
helm install stable/mysql --generate-name 
### 查看安装的 release
helm ls
### 查看 release 使用到的 values
helm get values releasename
### 更新
helm upgrade -f xxx.yaml mysql stable/mysql
### 查看历史版本
helm history releasename
### 回滚版本 默认是1
helm rollback releasename revision 

### 一般安装使用（更新也可以使用）推荐
1. 我们会先 fetch 下 chart 包到本地
2. 复制 values.yaml 参数到一个新的 yaml 文件，修改为自己想要的参数
3. 安装或更新，命令
helm upgrade --install flink-kubernetes-operator   --namespace itnio-flink --create-namespace . -f praise-dev-values.yaml
```



#### 内置对象变量

- `Release`：该对象描述了 release 本身的相关信息，它内部有几个对象： 

- `Release.Name`：release 名称
- `Release.Namespace`：release 安装到的命名空间
- `Release.IsUpgrade`：如果当前操作是升级或回滚，则该值为 true
- `Release.IsInstall`：如果当前操作是安装，则将其设置为 true
- `Release.Revision`：release 的 revision 版本号，在安装的时候，值为1，每次升级或回滚都会增加
- `Reelase.Service`：渲染当前模板的服务，在 Helm 上，实际上该值始终为 Helm

- `Values`：从 `values.yaml` 文件和用户提供的 values 文件传递到模板的 Values 值，默认情况下，Values 是空的。
- `Chart`：获取 `Chart.yaml` 文件的内容，该文件中的任何数据都可以访问，例如 `{{ .Chart.Name }}-{{ .Chart.Version}}` 可以渲染成 `mychart-0.1.0`，该对象下面可用的字段前面我们已经提到过了。

##### 

##### values

主要有3个来源：

- chart 文件中的 `values.yaml` 文件
- 用 `-f` 参数传递给 `helm install` 或 `helm upgrade` 的 values 值文件（例如 `helm install -f myvals.yaml ./mychart`）
- 用 `--set` 传递的各个参数（例如 `helm install --set foo=bar ./mychart`）



优先级： --set --> -f myvals.yaml --> 默认的values.yaml

###### 删除默认 KEY

如果你需要从默认值中删除 key，则可以将该 key 的值覆盖为 null

比如我们的values有这么一个配置

```yaml
livenessProbe:
  httpGet:
    path: /user/login
    port: http
  initialDelaySeconds: 120
```

想要让他变成

```yaml
livenessProbe:
  exec:
    command:
    - cat
    - docroot/CHANGELOG.txt
```

如果我们直接 用 `--set livenessProbe.exec.command=[cat, docroot/CHANGELOG.txt]`去覆盖的话是不行的，会变成

```yaml
livenessProbe:
  httpGet:
    path: /user/login
    port: http
  exec:
    command:
    - cat
    - docroot/CHANGELOG.txt
  initialDelaySeconds: 120
```

想要实现我们的目的，我们可以使用 `null`去删除他 `--set livenessProbe.exec.command=[cat, docroot/CHANGELOG.txt] --set livenessProbe.httpGet=null`



#### template

###### 函数和管道

quote 函数 字符串

upper 大写

repeat 重复次数

eq、ne、lt、gt、and、or 运算符

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ quote .Values.favorite.drink | repeat 5}}
  food: {{ quote .Values.favorite.food }}
  num1: {{  .Values.favorite.num1 }}
  num2: {{ quote .Values.favorite.num2 }}
  game: {{ .Values.favorite.game | upper | quote }}
```

###### 流程控制

控制流程为模板作者提供了控制模板生成流程的功能，Helm 的模板语言提供了以下一些流程控制：



-  `if/else` 条件语句 
-  空格控制 
-  `with` 指定一个作用域范围 
-  `range` 提供类似于 `for each` 这样的循环样式 



`if/else` 条件语句

```yaml
{{ if PIPELINE }}
  # Do something
{{ else if OTHER PIPELINE }}
  # Do something else
{{ else }}
  # Default case
{{ end }}
```

- 管道判断为 false 条件： 

-  布尔 false 
-  数字零 
-  一个空字符串 
-  nil（empty 或者 null） 
-  一个空集合（map、slice、tuple、dict、array） 

-  在其他条件下，条件都为真。 

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | default "tea" | quote }}
  food: {{ .Values.favorite.food | upper | quote }}
  {{ if eq .Values.favorite.drink "coffee" }}mug: true{{ end }}
```

```yaml
favorite:
  # drink: coffee
  food: pizza
```

空格控制



空格对于 YAML 文件非常重要的，不是说任意缩进就可以，依然还是以前面的例子为例，我们来格式化下模板格式以更易于阅读

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | default "tea" | quote }}
  food: {{ .Values.favorite.food | upper | quote }}
  {{ if eq .Values.favorite.drink "coffee" }}
    mug: true
  {{ end }}
```

这里我们运行  helm install --generate-name --dry-run --debug ./mychart 会报错

```shell
install.go:148: [debug] Original chart version: ""
install.go:165: [debug] CHART PATH: /Users/ych/devs/workspace/yidianzhishi/course/k8strain/content/helm/manifests/mychart

Error: YAML parse error on mychart/templates/configmap.yaml: error converting YAML to JSON: yaml: line 9: did not find expected key
```

因为我们生成的模版是这样的，明显的缩进有问题

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mychart-1575970308-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
    mug: true
```

去除多余的空格

```yaml
{{- if eq .Values.favorite.drink "coffee" }}
```

with 作用域



比如我们这个annotations的配置

```yaml
  {{- with .Values.forvorite.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
```

```yaml
ingress:
  enabled: true
  className: ""
  annotations: 
    kubernetes.io/ingress.class: nginx
    kubernetes.io/tls-acme: "true"
```

range 循环操作



类似于我们的for

```yaml
          env:
          {{- range $key, $value := .Values.env }}
          ## 如果 env 下是一个列表[] $key将会是索引号 0，1，2，如果 env下是一个{} $key 就是 {}里面的 key
          - name: {{ $key }}
            value: {{ $value | quote }}
          {{- end }}
          ports:
          {{- range .Values.service.ports }}
          - name: {{ .name }}
            containerPort: {{ .port }}
            protocol: TCP
          {{- end }}
```

```yaml
env:
  SPRING_PROFILES_ACTIVE: prod
  aaa: bbb
service:
  type: ClusterIP
  ports: 
    - port: 8080
      name: http
    - port: 7070
      name: http2
```

###### 变量

格式 `{{- $relname := .Release.Name -}}`

###### 命名模版

```yaml
{{/*
Common labels
*/}}
{{- define "test-demo.labels" -}}
helm.sh/chart: {{ include "test-demo.chart" . }}
{{ include "test-demo.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  labels:
  {{- include "test-demo.labels" . | nindent 4 }}
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | repeat 5 | quote }}
  food: {{ quote .Values.favorite.food }}
  num1: {{  .Values.favorite.num1 }}
  num2: {{ quote .Values.favorite.num2 |default "33" }}
  game: {{ .Values.favorite.game | upper | quote }}
```

```yaml
# Source: test-demo/templates/confipmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: test-demo-configmap
  labels:
    helm.sh/chart: test-demo-0.1.0
    app.kubernetes.io/name: test-demo
    app.kubernetes.io/instance: test-demo
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
data:
  myvalue: "Hello World"
  drink: "colecolecolecolecole"
  food: "banana"
  num1: 11
  num2: 33
  game: "LOL"
```

