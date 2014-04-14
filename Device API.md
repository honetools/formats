# Sway device API

The Sway tool discovers compatible devices over Bonjour, looking for “_sway._tcp” service type. Devices that implement the protocol below are compliant with this service type and may advertise themselves as such.

The Sway protocol consists of a few implemented HTTP endpoints.



## Object tree

Request a tree of represented objects and their values.

	GET /v1/objects

	{
		"View controllers": [
			{
				"class": "SomeClass",
				"guid": "something",
				"child_objects": [],
				"parameters": [
					{
						"name": "something",
						"type": "int"
						"value": 123
					}
				]
			}
		],
		"Views": [
			{
				"class": …
			}
		]
	}



## Put values

	PUT /v1/classes/SomeClass/values
	[{
		"name": "parameter_name",
		"type": "float"
		"value": 1234.0
	}]

	PUT /v1/classes/SomeClass/values
	[{
		"name": "parameter_name~type",
		"value": {
			// this is unlikely: better done in themes files
			"ipad~landscape": 123,
			"android": 456,
			"default": 12,
			"customTheme": 56
		}
	}]



## PNG representation

	GET /v1/objects/<guid>/png_representation
