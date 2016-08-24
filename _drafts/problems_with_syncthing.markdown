---
title: Problems_with_syncthing
date: 2016-08-24 09:21:49.693000000 Z
---

[I5V5M] 16:46:46 INFO: Compacting database: leveldb/table: corruption on table-footer (pos=2146254): bad magic number [file=302836.ldb]
panic: leveldb/table: corruption on table-footer (pos=1920973): bad magic number [file=261155.ldb]
[I5V5M] 16:46:46 OK: Ready to synchronize SigitaAndPeterPrivateShare (read-write)
goroutine 1 [running]:
github.com/syncthing/syncthing/lib/db.(*Instance).checkGlobals(0xc0822b6070, 0xc0822948e0, 0x16, 0x20, 0xc08377c8d8)
	c:/jenkins/workspace/syncthing-release-windows/src/github.com/syncthing/syncthing/lib/db/leveldb_dbinstance.go:578 +0x8a3
github.com/syncthing/syncthing/lib/db.NewFileSet(0xc0822892e0, 0x16, 0xc0822b6070, 0xc083ac6b08)
	c:/jenkins/workspace/syncthing-release-windows/src/github.com/syncthing/syncthing/lib/db/set.go:104 +0x2b2
github.com/syncthing/syncthing/lib/model.(*Model).AddFolder(0xc0838d0000, 0xc0822892e0, 0x16, 0xc0822989a0, 0x8, 0xc0821ee780, 0x3, 0x4, 0x0, 0x15180, ...)
	c:/jenkins/workspace/syncthing-release-windows/src/github.com/syncthing/syncthing/lib/model/model.go:1211 +0x14f
main.syncthingMain(0x0, 0x0, 0x0, 0x0, 0x0, 0x1, 0xc082007180, 0x34, 0x0, 0xc08205400d, ...)
	c:/jenkins/workspace/syncthing-release-windows/src/github.com/syncthing/syncthing/cmd/syncthing/main.go:690 +0x1fd6
main.main()
	c:/jenkins/workspace/syncthing-release-windows/src/github.com/syncthing/syncthing/cmd/syncthing/main.go:362 +0x94f

solution: delete in %LOCALAPPDATA%\Syncthing the index-... folder afterwards everything works