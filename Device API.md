# Sway device API

The Sway tool discovers compatible devices over Bonjour, looking for “_sway._tcp” service type. Devices that implement the protocol below are compliant with this service type and may advertise themselves as such.

The Sway protocol consists of a few implemented HTTP endpoints.


## Device guid

PUT /v1/device_session_start

Content:
{
	device_talkback_url: http://something:12345/v1/device_talkback
}

Response: same as device_info below

GET /v1/device_info

Returns device GUID and name. GUID must remain stable as long as this app session is running, even if it goes to background and comes to foreground several times.

{
	device_name: "John Appleseed’s iPhone",
	device_guid: "8746A183-6470-496C-8F62-45396A13B353",
	app_id: "12345" # Sway app identifier
}


## Object tree

Request a tree of represented objects and their values. JSON structure is the same as in the document.

	GET /v1/objects

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




## Put values

	PUT /v1/objects/SomeClass/parameters
	[{
		"parameter_name~float": 123.5,
	},
	{
		"parameter~string": "hello"
	}
	}]


	PUT /v1/objects/SomeClass/parameters/themeName
[{
	"parameter_name~float": 123.5,
},
{
	"parameter~string": "hello"
}
}]


## PNG representation

	GET /v1/objects/<guid>/png_representation

  GET /v1/png_representation


# Sway device talkback API

PUT /v1/device_talkback

Receive an array of log entries (even if just one) from device about values being used in real time

[{
	time: 1231233128, # timestamp
	device_guid: guid,
	object: objectClass,
	parameter:, objectParameterName,
	parameter_type: type.
	theme: themeName,
	value: currentValue
}]

returns: 200 OK
