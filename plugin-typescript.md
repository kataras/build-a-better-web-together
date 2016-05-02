# Typescript

[This is a package](https://github.com/kataras/iris/tree/development/plugin/typescript)


 1. Bin: string, the typescript installation path/bin/tsc or tsc.cmd, if empty then it will search to the global npm modules
 2. Dir: string, Dir set the root, where to search for typescript files/project. Default "./" 
 3. Ignore: string, comma separated ignore typescript files/project from these directories. Default "" (node_modules are always ignored) 
 4. Tsconfig: &typescript.Tsconfig{}, here you can set all compilerOptions if no tsconfig.json exists inside the 'Dir' 
 5. Editor: typescript.Editor(), if setted then alm-tools browser-based typescript IDE will be available. Defailt is nil