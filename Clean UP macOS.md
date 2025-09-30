- https://gemini.google.com/app/5d6dc2fabd9feaf0

```
df -h
Filesystem        Size    Used   Avail Capacity iused ifree %iused  Mounted on
/dev/disk3s5     460Gi   372Gi    67Gi    85%    3.8M  706M    1%   /System/Volumes/Data


find ~ -size +100M -print


cd ~/Library/Caches/
du -sh * | sort -hr

 29G	JetBrains
4.9G	Google
3.1G	Homebrew
1.3G	Yarn
1.3G	colima
656M	go-build
538M	node-gyp
525M	pip
444M	SiriTTS
412M	cabos-updater
405M	lima
375M	podman-desktop-updater
338M	Microsoft Edge
284M	Firefox
176M	com.oracle.java.JavaAppletPlugin
175M	vscode-cpptools
114M	next-swc
109M	snyk
104M	ShiftData
```