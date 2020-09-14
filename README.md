## Go-Modules

#### Init
go mod init github.com/jt/gomodules
Result: creates _go.mod_ file with our module's name and suppported golang version

#### Adding External Dependencies
`go get -u github.com/gorilla/mux`  # downloads latest repository
go.mod:
require github.com/gorilla/mux v1.8.0 // indirect
Also updates go.sum with sha1 value

```
$ go list

github.com/jt/gomodules
```

```
$ go list -m all # lists all modules those are involed with our modules (and not bare packages)

github.com/jt/gomodules

github.com/gorilla/mux v1.8.0
```

```
$ go list -m -versions github.com/gorilla/mux

github.com/gorilla/mux v1.2.0 v1.3.0 v1.4.0 v1.5.0 v1.6.0 v1.6.1 v1.6.2 v1.7.0 v1.7.1 v1.7.2 v1.7.3 v1.7.4 v1.8.0
```

```
$ go list -m -versions github.com/gorilla/context

github.com/gorilla/context v1.1.1
```

#### Managing Dependencies

```
# verifies the hash of go modules downloaded, 
# if somehow the hash is changed may be by man in the middle
# attack or manual change in go/pkg/mod then verify will fail. 
# manually resetting modules does not work as hash is generated with time component.
# The only way to get it working by once again downloading go dependency - go get ...
$ go mod verify 

all modules verified
```

```
# Remove unused modules 
go mod tidy 
```

#### Versioning
v - (required) version prefix

1 - (required) Major version - likely to break backward compatibility 

2 - (required) Minor verison - adds new features, doesn't remove existing, doesnt break backward compatibility

3 - (required) Patch version - adds bug fixes, doesnt break backward compatibility

pre1 - (if required) to indicate pre-release of new version (arbitrary text)

```
Convension:

alpha < beta
pre1 < pre2

(usually follow alphabetic order)
```

#### Versioning Rules

 - v1 and earlier

    + No backword compatibiliry is expected
    + the apis are not stable and they are subject to change at any point in time.
    + go tooling assume the same
    + `import github.com/gorilla/mux` 

```
# Listing every versio of a quote modules

$ go list -m -versions rsc.io/quote
rsc.io/quote v1.0.0 v1.1.0 v1.2.0 v1.2.1 v1.3.0 v1.4.0 v1.5.0 v1.5.1 v1.5.2 v1.5.3-pre1
$ go list -m -versions rsc.io/quote/v2
rsc.io/quote/v2 v2.0.0 v2.0.1
$ go list -m -versions rsc.io/quote/v3
rsc.io/quote/v3 v3.0.0 v3.1.0


$ go get rsc.io/quote
go: rsc.io/quote upgrade => v1.5.2 # downloads latest of v1.x

$ go get rsc.io/quote/v2 # downloads latest of v2.x
go: downloading rsc.io/quote/v2 v2.0.1
go: rsc.io/quote/v2 upgrade => v2.0.1 

$ go get rsc.io/quote/v3 # downloads latest of v3.x
go: downloading rsc.io/quote/v3 v3.1.0
go: rsc.io/quote/v3 upgrade => v3.1.0

# Keeps all versions of the quote modules in 'go.mod' file
# Allowing multiple versions of the module ensure silent/breake-free  migration of module from one version to another
```

 - v2+

    + Backward compatibility is preserved within a major version when minor version and/or patch version is upgraded
    + If backward compatibility (BC) should be broken then consider increasing major version
    + Each major version (v2+) has unique import path e.g. `import github.com/gorilla/mux/v2`
    + both v2- and v2+ dependencies can be imported at the same time in project.


 - unversioned

    +  Can use prerelease identifier, it should be in alphanumneric order, such as a, b, c. 'a' has less precedence than 'b' and so on. Additionally Timestamp confirms the cronological order and commithash is used as an another additional way to uniquely identify the module version that we pull in. Such as 

    require golanf.org/x/tools v0.0.0-20180917221912-90fa456767

#### Module Queries 

- #### specific version: 
  - `@v1.7.2` 
``` 
$ go get github.com/gorilla/mux@v1.6.1
go: downloading github.com/gorilla/mux v1.6.1

$ go get github.com/gorilla/mux@master
go: github.com/gorilla/mux master => v1.8.1-0.20200912192056-d07530f46e1e
go: downloading github.com/gorilla/mux v1.8.1-0.20200912192056-d07530f46e1e

# in go.mod 
# v1.8.1 does not exist yet but go tooling presume that the next succesive release after `v1.8.0` would be `v1.8.1` and to add more uniqueness it adds timestamp and commithash
github.com/gorilla/mux v1.8.1-0.20200912192056-d07530f46e1e // indirect

# downloads the latest released version
$ go get github.com/gorilla/mux@latest
go: github.com/gorilla/mux latest => v1.8.0

```

- Version prefix: `@v1` allows any minor and patch version
- Latest: `@latest` pulls the latest released version
- Specific commit: `@456c34e`
- Specific commit or tree or branch: `@master`
- #### Comparison: 
  - `@>=1.7.3`

```
# The comparison rules does not preserve the query, in go.mod file it pulls specific version which is the closest match to query

$ go get "github.com/gorilla/mux@<v1.8.0"
go: github.com/gorilla/mux <v1.8.0 => v1.7.4

$ go list -m -versions github.com/gorilla/mux
github.com/gorilla/mux v1.2.0 v1.3.0 v1.4.0 v1.5.0 v1.6.0 v1.6.1 v1.6.2 v1.7.0 v1.7.1 v1.7.2 v1.7.3 v1.7.4 v1.8.0

# in go.mod
github.com/gorilla/mux v1.7.4 // indirect
```

- #### Comparison Rules: 
  - Closest match wins: for reproducible builds. e.g. if set is `@>=1.7.3` and avaoilable is `v1.7.3` , `v1.7.4` and `v1.7.5`, then the set rule matches with `v1.7.3` 
  - Prerelease versions have lower precedence: v2-pre1 has lower precedence over v1.x

#### Advanced
- ##### `go mod why <module>`
  - Tells us why and what's the purpose of this module 

```
# 'go mod why' is resource intensive as it builds modules' dependency graph, hence to be avoided in ci/cd pipeline
# Response when the module is unused
$ go mod why github.com/gorilla/mux
# github.com/gorilla/mux
(main module does not need package github.com/gorilla/mux)

# Response when module is used in source code
$ go mod why github.com/gorilla/mux
# github.com/gorilla/mux
github.com/jt/gomodules
github.com/gorilla/mux
```

- ##### `go mod graph`
  - Creates a module dependency graph of the module you are in
```
$ git clone https://github.com/rsc/quote.git
Cloning into 'quote'...
remote: Enumerating objects: 84, done.
remote: Total 84 (delta 0), reused 0 (delta 0), pack-reused 84
Unpacking objects: 100% (84/84), done.
$ cd quote/
$ go mod graph
rsc.io/quote rsc.io/quote/v3@v3.0.0
rsc.io/quote rsc.io/sampler@v1.3.0
rsc.io/quote/v3@v3.0.0 rsc.io/sampler@v1.3.0
rsc.io/sampler@v1.3.0 golang.org/x/text@v0.0.0-20170915032832-14c0d48ead0c
```
- ##### `go mod edit <module>`
  - Programmatically changes the _go.mod_
  - -require, -droprequire, -exclude, -dropexclude, -replace, -dropreplace

```
   # Changes our module name to 'github.com/jt/mod' in go.mod
   $ go mod edit -module github.com/jt/mod

   # Changes our go version in go.mod
   $ go mod edit -go 1.12

   $ go mod edit -require github.com/gorilla/mux@v1.1.1 # pulls v1.1.1
   $ go mod edit -droprequire github.com/gorilla/mux # removes gorilla/mux

   $ go mod edit -replace rsc.io/quote@v1.5.2=../quote # replaces the quote@v1.5.2 module by a local rsc.io/quote module but keeps original go module dependency in require block.

   $ go mod edit -dropreplace rsc.io/quote@v1.5.2 # removes replacement 
```

- #### `go mod download <module@version>`
  - Downloads the specific dependency into local machine cache and can be pulled directly from local cache.

```
  $ go mod download rsc.io/quote/v3@v3.0.0

  $ go mod download # pre-fills the local cache or to compute the answers for a Go module proxy

  $ go mod download -json rsc.io/quote/v3@v3.0.0 
{
        "Path": "rsc.io/quote/v3",
        "Version": "v3.0.0",
        "Info": "/home/user/go/pkg/mod/cache/download/rsc.io/quote/v3/@v/v3.0.0.info",
        "GoMod": "/home/user/go/pkg/mod/cache/download/rsc.io/quote/v3/@v/v3.0.0.mod",
        "Zip": "/home/user/go/pkg/mod/cache/download/rsc.io/quote/v3/@v/v3.0.0.zip",
        "Dir": "/home/user/go/pkg/mod/rsc.io/quote/v3@v3.0.0",
        "Sum": "h1:OEIXClZHFMyx5FdatYfxxpNEvxTqHlu5PNdla+vSYGg=",
        "GoModSum": "h1:yEA65RcK8LyAZtP9Kv3t0HmxON59tX3rD+tICJqUlj0="
}
```
- #### `go mod vendor`
  - Creates a `vendor` dir and downloads all dependent modules in it, the use case would be not reaching to internet while building the application when deployed in the production
  - It is used in case your project is inside the workspace (old style of golang project development)
  - 
```
$ go mod vendor # analyses the module dependency and creates a vendor directory with all dependency, it should not be confsed with module cache.

# if the project is outside of workspace and needs to use the dependency from 'vendor' directory then run the project with 

$ go run -mod=vendor

# or build an executable 
$ go run -mod=vendor
$ ./gomodules
2020/09/14 17:29:11 Hello Gomodules
Patched in vendor!!
``` 

- #### `go mod readonly`
  - keeps the _go.mod_ readonly and go tooling does not change by its own 
 ```
 go build -mod=readonly .
 ```
