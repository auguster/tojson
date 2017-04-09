# tojson: converts stdin to JSON
The JSON fields are read from the first line read from stdin. If you want to use your own fields you can pass them as parameters of `tojson`. In that case the first line of stdin will be considered as data.

If the input lines that contain more arguments than the number of fields are merged into an array into the last field.

## Syntax
`cmd | ./tojson [field name separated by spaces]`

If you want to read from a file, you can of course `cat file | ./tojson`. Because `tojson` handles continuous input you can convert a file on the fly.

Once you have your JSON output you can output it to a file or pipe it to `jq '.'`. The later can be used for filtering or color output.

## Examples
Here are some examples of increasing complexity.

### Simple
`ps | ./tojson`

### Last field array
`ps -f | ./tojson`

In this example, the input sometime contains more arguments than estimated fields. This extra data is merged into the last field as an array.

### Using custom fields
If the input has field name containing space (or more than one word per field), you can specify your own field name. This is especialy useful if you want to translate or impose specific field name.

In my case `df` first line is `Sys. de fichiers Taille Utilisé Dispo Uti% Monté sur` (french bash). `Sys. de fichiers` and `Monté sur` should each be one field and the accents might break JSON parsing. The solution here is the provide `tojson` with the field name. In that case the first output of `tojson` will the column name, you can drop it from the input or the output easily.

For example:

`df | ./tojson device size used available free mount |tail -n +2`

### Continuous conversion
You can convert a log file as it's being written.

`vmstat -n 1 > /tmp/vmstat` will write a new line every second. You could pipe the command directly to `tojson` but you can also read the file on the fly with `tail -f /tmp/vmstat |./tojson $(vmstat |sed -n 2p)`.

## Making a simple server
Lets serve the output of `vmstat -n 1` as JSON on a simple server.

`vmstat -n 1 |./tojson $(vmstat |sed -n 2p) |while read json; do echo "$json" > vmstat.json; done`

Will update `vmstat.json` every second. To serve the file you can use any simple HTTP server. A list of simple HTTP server can be found in this [gist](https://gist.github.com/willurd/5720255).
For example: `busybox httpd -f -p 8080`
Let's go one step further and use docker: `docker run -d -v $PWD/vmstat.json:/vmstat.json -p 8080 busybox busybox httpd -f -p 8080`.
