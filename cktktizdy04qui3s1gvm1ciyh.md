## Vercel's pkg - Package your Node.js app into a single executable

Hello all, In this short post, we will take a look at Vercel's pkg - how we can package the Node.js app into a single executable. 

When we are developing a Node.js application, it totally makes sense to install dev tools, dependencies to build and run an application. In the end, the user is interested in running the app and not the code and libraries the developer integrates. In the Node.js world, it's getting tougher that - when we release the packages, the sources are also released along with the 100s of dependency code as node_modules. It brings the necessity of the package mechanism to build and distribute the node library as a single executable - with the runtime or as a single file to run on already installed nodejs runtime.

### Build the Node.js app into a single file - ncc

%[https://github.com/vercel/ncc]

ncc - Simple CLI for compiling a Node.js module into a single file, together with all its dependencies, gcc-style.

Install the ncc using the below command.

```docker
npm i -g @vercel/ncc
```

 Build the project using this simple command. It will output the Node.js compact build of app.js into dist/app.js

```docker
ncc build app.js -o dist
```

### Package the Node.js app into a single executable - pkg

%[https://github.com/vercel/pkg]

pkg - This command-line interface enables you to package your Node.js project into an executable that can be run even on devices without Node.js installed.

Install the pkg using the below command

```docker
npm install -g pkg
```

Run the pkg build targeting multiple platforms. It will create the executable in the dist directory

```docker
pkg -t node12-linux,node14-linux,node14-win index.js
```

**Targets**

`pkg` can generate executables for several target machines at a time. You can specify a comma-separated list of targets via `--targets` option. A canonical target consists of 3 elements, separated by dashes, for example `node12-macos-x64` or `node14-linux-arm64`:

- **nodeRange** (node8), node10, node12, node14, node16 or latest
- **platform** alpine, linux, linuxstatic, win, macos, (freebsd)
- **arch** x64, arm64, (armv6, armv7)

### Demo

 Check out this repo 

%[https://github.com/ksivamuthu/vercel-pkg-demo]

Run locally and verify whether it's logging the telemetry in the console.

```docker
➜ vercel-pkg-demo git:(main) npm i
➜ vercel-pkg-demo git:(main) node index.js
IoT Device Initialized
Telemetry: {"temperature":"56.12","humidity":"52.11"}
Telemetry: {"temperature":"68.30","humidity":"51.90"}
Telemetry: {"temperature":"53.78","humidity":"53.72"}
```

Run `npm run build` to build the package into executable.

```docker
"scripts": {
    "build": "npx pkg -t linux,macos,win . --out-path dist"
 },
```

The build steps are added in GitHub Actions and you can see the executable files are added as artifacts after build.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1631668176228/CYq3OOsGM.png)

Let's download the executable and run and see whether we are seeing the same output.

```docker
./hvac-demo-device-macos
➜  dist git:(main)  ./hvac-demo-device-macos
IoT Device Initialized
Telemetry: {"temperature":"70.59","humidity":"46.23"}
Telemetry: {"temperature":"69.76","humidity":"49.93"}
Telemetry: {"temperature":"62.15","humidity":"58.93"}
```

### Use cases

- Make a commercial version of your application without sources
- Make a demo/evaluation/trial version of your app without sources
- Instantly make executables for other platforms (cross-compilation)
- No need to install Node.js and npm to run the packaged application
- Put your assets inside the executable to make it even more portable

### Conclusion

I've found vercel's pkg very useful in order to achieve a similar target and package a whole application into standalone executables for multiplatform. It's nice to have a single file that can be started right away without any external dependency. And also, it prevents from having to distribute the full sources. You can extend it for including assets and other requirements.

I'm Siva - working as Sr. Software Architect at Computer Enterprises Inc from Orlando. I'm an AWS Community builder, Auth0 Ambassador and I am going to write a lot about Cloud, Containers, IoT, and Devops. If you are interested in any of that, make sure to follow me if you haven’t already. Please follow me @ksivamuthu Twitter or check out my blogs at blog.sivamuthukumar.com