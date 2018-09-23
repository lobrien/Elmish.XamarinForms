Fabulous - Guide
=======

{% include_relative contents.md %}

Models
--------

### Messages and Validation

Validation is generally done on updates to the model, storing error messages from validation logic in the model
so they can be correctly and simply displayed to the user.  Here is an example of a typical pattern.

```fsharp

    type Temperature = 
       | Value of double
       | ParseError of string  
       
    type Model = 
        { TempF: Temperature
          TempC: Temperature }

    /// Validate a temperature in Farenheit, can be shared between client/server
    let validateF text =  ... // return a Result

    /// Validate a temperature in celcius, can be shared between client/server
    let validateC text = // return a Result 

    let update msg model =
        match msg with
        | SetF textF -> 
            match validateF textF with
            | Ok newF -> { model with TempF = Value newF }
            | Error msg -> { model with TempF = ParseError msg }
            
        | SetC textC -> 
            match validateC textC with
            | Ok newC -> { model with TempC = Value newC }
            | Error msg -> { model with TempC = ParseError msg }
```

Note that the same validation logic can be used in both your app and a service back-end.

### Saving Application State

Application state is very simple to save by serializing the model into `app.Properties`. For example, you can store as JSON as follows using `Json.NET`:
```fsharp

type Application() = 
    ....
    let modelId = "model"
    override __.OnSleep() = 

        let json = Newtonsoft.Json.JsonConvert.SerializeObject(runner.CurrentModel)
        Debug.WriteLine("OnSleep: saving model into app.Properties, json = {0}", json)

        app.Properties.[modelId] <- json

    override __.OnResume() = 
        Debug.WriteLine "OnResume: checking for model in app.Properties"
        try 
            match app.Properties.TryGetValue modelId with
            | true, (:? string as json) -> 

                Debug.WriteLine("OnResume: restoring model from app.Properties, json = {0}", json)
                let model = Newtonsoft.Json.JsonConvert.DeserializeObject<App.Model>(json)

                Debug.WriteLine("OnResume: restoring model from app.Properties, model = {0}", (sprintf "%0A" model))
                runner.SetCurrentModel (model, Cmd.none)

            | _ -> ()
        with ex -> 
            App.program.onError("Error while restoring model found in app.Properties", ex)

    override this.OnStart() = 
        Debug.WriteLine "OnStart: using same logic as OnResume()"
        this.OnResume()
```

