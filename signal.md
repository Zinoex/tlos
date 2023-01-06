# Available modes

Get a list of available control modes, i.e. what lights can be lit simultaneously.

**Topic** : `/signal/modes/`

**Method** : `GET`

```json
{
    "type": "request",
    "topic": "/signal/modes/",
    "method": "GET"
}
```

## Responses

```json
{
    "type": "response",
    "topic": "/signal/modes/",
    "method": "GET",
    "status": 200,
    "payload": [
        {
            "mode": "uuid",
            "mode_type": "type"
        },
        {
            "mode": "uuid",
            "mode_type": "type"
        },
        {
            "mode": "uuid",
            "mode_type": "type"
        }
    ]
}
```