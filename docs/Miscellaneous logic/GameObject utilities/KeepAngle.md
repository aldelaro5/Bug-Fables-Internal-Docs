# KeepAngle
A component meant to be attached on any GameObject via Unity.

It will enforce on every LateUpdate the angles of the GameObject to stay at the `angle` field which can be configured in the inspector. There is also a `getatstart` configuration field that if set to true, it will cause `angle` to be set to the starting angle of the GameObject on Start and to use it instead of the inspector value on LateUpdate.
