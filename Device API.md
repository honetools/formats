# Sway device API

The Sway tool discovers compatible apps devices over Bonjour, looking for “_sway._tcp” service type. It then uses the protocol to get info about the apps running on devices, and to set the parameter values. Devices that implement the protocol below are compliant with this service type and may advertise themselves as such. This protocol is fully implemented by the official Sway libraries for iOS and OS X that you include in your app.

The Sway protocol consists of a few implemented HTTP endpoints.


### Device info

The tool requests info about the device with this call.

Request: `GET /v1/device_info`

Response:

		{
			device_name: "John Appleseed’s iPhone",
			device_guid: "8746A183-6470-496C-8F62-45396A13B353",
			project_id: "12345",
			project_name: "com.example.myGreatApp"
		}

`device_name`: a string that helps the user to identify the device that the app is running on. This could be a combination of device name and app name.

`device_guid`: A stable identifier that is unique for the combination of this device, app, and Sway. The Sway library generates a unique persistent identifier that remains stable every time the app is run.

`project_id`: this project’s Sway identifier.

`project_name`: this project’s bundle name or some other kind of name that helps the user to identify the project.



### Device session start

The Sway tool uses the device_session_start call to inform the device about its presence, and give the app the talkback URL that the app may use to send info to the tool using the Sway device talkback protocol.

The device responds with the same info as device_info call.

Request: `PUT /v1/device_session_start`

Request body:

		{
			device_talkback_url: http://something:12345/v1/device_talkback
		}

Response: same as device_info above



### Object tree

Returns a tree of Sway objects and parameter values that is known to the app. The data structure is the same as in Sway documents, but represented as JSON instead of YAML.

Request: `GET /v1/objects`

Response:

		[
			{
				"someObject":
				[{"parameter1~float":23.4},{"parameter2~string":"hello"}]
			},
			{
				"anotherObject":
				[{"parameter3~float":23.4},{"parameter4~string":"hello"}]
			}
		]



### Put parameter values

The tool puts new parameter values to the device with this call. The theme is indicated by the URL. If no theme suffix is present, default theme is assumed.

Request (default theme): `PUT /v1/objects/SomeObject/parameters`

Request (some other theme): `PUT /v1/objects/SomeObject/parameters/themeName`

Request body:

		[{
			"parameter_name~float": 123.5,
		},
		{
			"parameter~string": "hello"
		}
		}]



### App PNG representation

The tool requests the current visual representation of the app as a PNG file. What’s an appropriate representation is left to the device side to decide. In case of devices, this is obviously what’s currently represented on the device screen. In case of desktop applications, this could be the single application window, or some form of different windows composited/laid out into a single PNG.

Request: `GET /v1/png_representation`

Response: the PNG image



## Sway device talkback API

Whereas the device API is implemented on the device side, the device talkback API is implemented in the Sway tool. Apps implementing Sway can talk back to the Sway tool with the talkback API. They discover the tool endpoint URL with the device API device_session_start call when the tool informs the device about its presence and communicates the device talkback URL to the device.

Request: `PUT /v1/device_talkback`

Request JSON body:

		[{
			time: 1231233128,
			device_guid: guid,
			project_id: project_id,
			object: objectClass,
			parameter: objectParameterName,
			parameter_type: type,
			theme: themeName,
			value: currentValue,
			error: "Error message"
		}]

* `time`: the UNIX timestamp of the value access
* `parameter_type`: one of `int`, `float`, `color`, `string` or `font`.
* `theme`: named theme, or empty string "" for default theme.
* `value`: serialized parameter value

If all is OK, "error" key is missing. If there is a problem with a value, "error" key is present with the text.

returns: 200 OK
