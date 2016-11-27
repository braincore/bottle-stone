# Bottle-Stone Plugin

This plugin makes it easy to use the [Stone Interface Description Language]
(https://github.com/dropbox/stone) with the [Bottle Web Framework]
(https://github.com/bottlepy/bottle).

Arguments and results to routes are encoded as JSON in the HTTP body.

## Install

    pip install bottle-stone

## Register Plugin

    import bottle
    import bottle_stone
    bottle.install(bottle_stone.BottleStonePlugin(GEN_PKG_PATH))

`GEN_PKG_PATH` is the Python (sub-)package that contains the output of an
invocation of the Stone "python_types" generator.

    $ stone python_types GEN_PKG_PATH [spec [spec ...]]

Note that the output of python_types isn't a package because it does not
generate an `__init__.py` file. You'll have to do so manually:

    $ touch GEN_PKG_PATH/__init__.py

## Using

The `name` specified in your route decorator must conform to a specific format:

    @route(ROUTE_PATH, name='{namespace}.{route_name}')

Here's an example where a namespace `articles` is defined in Stone with a route
called `list`:

    @route('/1/articles/list', name='articles.list')
    def articles_list(arg):
        ...
        return ListResult(...)  # this must be imported from GEN_PKG_PATH

`arg` will be a Stone-generated Python object representing the argument to the
route. The route handler will need to return the Stone-generated Python object
that is the result of the route. Return `None` if the result data type is
`Void`.

To return an error, you'll need to raise an `ApiError` exception:

    from bottle_stone import ApiError

    @route('/1/articles/list', name='articles.list')
    def articles_list(arg):
        ...
        raise ApiError(ListError(...))

## HTTP Status Codes

HTTP requests whose bodies do not conform to the Stone specification are
rejected with a `400 Bad Request` error. When an `ApiError` is raised, the HTTP
response code is by default `418 I'm a teapot`, but it can be changed to an
arbitrary status code via the `http_error_status_code` argument to
`BottleStonePlugin`.
