# Typescript scale host

The typescript scale host can be used either as a node library, or in a browser context.
The scale host allows you to run scale modules.

A scale function takes in a Context and returns a Context or an Error.

## Run tests

`npm run test`

## Build

`npm run build`

## Using the typescript scale host

Here's a simple example showing usage.
To use scale functions, you must first decide on a signature or use a pre-defined signature such as `scale-signature-http`.

First step, we setup a signatureFactory to create new contexts.
A context encapsulates a set of inputs and outputs to the scale function. So for example, in the `scale-signature-http` we would have `Request` and `Response` objects, with the usual HTTP components - headers, body, etc.
In this example, our context just has a single `Data` object defined.

    const signatureFactory = () => {
      return new Context();
    }

Next we load up the scale function we wish to use. This is in the form of a wasm file, and some metadata.
Scale functions can be written in any language, but must be for the correct signature.
We can use many scale functions, and one scale function can call the next by calling `next`. This should be familiar to anyone having written http middleware for example.

    // Load a wasm module
    const modPassthrough = fs.readFileSync("./src/runtime/modules/passthrough-TestRuntime.wasm");

    // Define the scale function we are using
    const scalefnPassthrough = new ScaleFunc(V1Alpha, "Test.Passthrough", "ExampleName@ExampleVersion", Go, [], modPassthrough);

Now we have a scale function, and a signature factory, we need to get a runtime, and then wait for a specific instance.

    // Ask for a runtime, using the signatureFactory, and the scale function.
    const r = await GetRuntime(signatureFactory, [scalefnPassthrough]);

    // Wait for an instance
    const i = await r.Instance(null);

Now we can use the instance. First we set any context data.
Then we call `i.Run()` (Note that this can throw errors).
Then we look at any output data in the context.

    // Set any context input data
    i.Context().Data = "Test Data";

    // Run the scale function
    i.Run();

    // Get any output from the function.
    const output = i.Context().Data;

