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
