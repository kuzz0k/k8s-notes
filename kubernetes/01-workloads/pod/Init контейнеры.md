Это контейнеры которые запускаются перед запуском основных. При удачном запуске init контейнера далее будет запускаться основной.
```
spec:
	initContainers:
		- name: init-myservice
		image: busybox:1.28
		command:
			- 'sh'
			- '-c'
			- 'echo The app is running!; sleep 5'
		resources:
			requests:
				memory: "64Mi"
				cpu: "250m"
			limits:
				memory: "128Mi"
				cpu: "500m"
```
