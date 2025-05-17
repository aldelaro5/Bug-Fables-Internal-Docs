
```cs
public void DoClock()
```
Perform tasks once an in game timer clock tick has passed which is normally every second (the method is designed to be called using InvokeRepeating every 1.0 second which is typically setup by EventControl).

TODO: possibly document this system separately, there's a lot of important stuff going on here notably, the Resources.UnloadUnusedAssets calls spams
