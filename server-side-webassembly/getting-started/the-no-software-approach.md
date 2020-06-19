---
description: Get started with Rust functions in Node.js app without installing any software
---

# VS Codespaces

## This page is outdated. Please [visit here for the most up-to-date content](https://www.secondstate.io/articles/why-webassembly-server/).

In the [previous tutorial](./), we discussed how to create and access Rust functions in Node.js apps. In this tutorial, I will show you how to experiment with development without installing any developer tools software.

![https://www.youtube.com/watch?v=j85cbNsciOs](../../.gitbook/assets/ssvm-edited-without-music.gif)

\*\*\*\*[**Fork this GitHub repository**](https://github.com/second-state/ssvm-nodejs-starter/fork) to get started. In your fork, you can use GitHub's web UI to edit source code files.

* The Rust files are in the `src` directory. You can put high performance workload into Rust functions. The Rust build and dependency configuration is in the `Cargo.toml` file.
* The JavaScript files are in the `node` directory and they can access the Rust functions.
  * The `node/app.js` file contains the application.

### VS Codespaces steps

{% hint style="info" %}
VS Codespaces runs entirely in your browser and costs around $1 per work day. It is cheaper than a cup of coffee in the office. Alternatively, in steps 1-2 below, you could use locally installed VSCode and Docker, and [launch the IDE with your remote git repository](https://code.visualstudio.com/remote-tutorials/containers/getting-started).
{% endhint %}

First, open the [VSCode Codespaces](https://online.visualstudio.com/) web site and login with your Azure account. You can get a [free Azure account](https://azure.microsoft.com/en-us/free/).

Next, create a new codespace. Put your forked repository into the Git Repository field. This step takes a few minutes. But once a codespace is created, subsequent openings only take seconds.

![](../../.gitbook/assets/vscode_create.png)

Open the source code files and make changes as you wish!

![](../../.gitbook/assets/vscode_code.png)

Click on the Run button on the left panel, and then the Launch Program at the top to build and run the application.

![](../../.gitbook/assets/vscode_run.png)

The Terminal window at the bottom shows the build progress. It builds the Rust program, and then launches the Node.js app.

![](../../.gitbook/assets/vscode_build.png)

The Debug window shows the Node.js server running and waiting for web requests.

![](../../.gitbook/assets/vscode_debug.png)

Now, open another terminal window in the IDE via the `Terminal -> New Terminal` menu.

![](../../.gitbook/assets/vscode_terminal.png)

From the terminal window, you can test the local server.

```text
$ curl http://127.0.0.1:3000/?name=SSVM
hello SSVM
```

In fact, you can run any Linux command from VSCode's built-in Terminal. You could run `ssvmup build` to build, and then `node node/app.js` to run the Node.js application. The Node.js application could be a server application as we have shown here, or simply a command line program as many of our [later examples](../pass-any-argument-and-return-any-value.md).

![](../../.gitbook/assets/vscode_terminal_ssvm.png)

That's it! VSCode has many useful features such as real time error detection and syntax highlighting as you type code, advanced Github integration, and integrations with many many development tools. Enjoy coding!

