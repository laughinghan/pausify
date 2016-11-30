# Pause and resume serving requests

Example, pausing an HTTP server:

    function HTTP_Example () {
      var pausify = require('pausify')
      var helloWorld = pausify(function (req, res) {
        res.end('Hello world\n')
      })
      require('http').createServer(helloWorld).listen(1337)
      // ...
      something.on('change', function () {
        helloWorld.pause()
        regenerateStuff().on('end', helloWorld.resume)
      })
    }

Example, pausing TCP port forwarding from 8000 &rarr; 1337:

    function TCP_Example () {
      var pausify = require('pausify'), net = require('net')
      var fwd = pausify(function (incoming) {
        incoming.pipe(net.createConnection({ port: 1337 }).pipe(incoming))
      })
      net.createServer(fwd).listen(8000)
      // ...
      something.on('change', function () {
        fwd.pause()
        regenerateStuff().on('end', fwd.resume)
      })
    }

How? Easy:

    module.exports = function (serve) {
      var q
      function pauseOrServe () {
        if (q) q.push(this, arguments)
        else serve.apply(this, arguments)
      }
      pauseOrServe.pause = function () {
        if (!q) q = []
      }
      pauseOrServe.resume = function () {
        for (var i = 0; i < q.length; i += 2) serve.apply(q[i], q[i+1])
        q = undefined
      }
      return pauseOrServe
    }