title: t02. Webpack&Socketio
speaker: LI YANG
url: 
transition: slide3
files: /js/demo.js,/css/demo.css
theme: dark
usemathjax: yes

--
### WebPack打包socket.io
webpack.config.js

```
var dist_dir = __dirname + '/dist';
 
var nodeExternals = require('webpack-node-externals');
 
module.exports = [{
    target: 'node',
    externals: [nodeExternals()],
    entry: {
        app: "./src/server.ts"
    },
    output: {
        path: dist_dir,
        filename: "server.js"
    },
    resolve: {
        extensions: ['', '.js', '.ts']
    },
    module: {
        loaders: [
            { test: /\.ts$/, loader: "ts-loader", exclude: /node_modules/ }
        ]
    }
}];

```
