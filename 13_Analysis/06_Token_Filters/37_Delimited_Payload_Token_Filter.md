## Delimited Payload Token Filter

Named `delimited_payload_filter`. Splits tokens into tokens and payload whenever a delimiter character is found.

Example: "the|1 quick|2 fox|3" is split by default into tokens `the`, `quick`, and `fox` with payloads `1`, `2`, and `3` respectively.

Parameters:

`delimiter`
     Character used for splitting the tokens. Default is `|`. 
`encoding`
     The type of the payload. `int` for integer, `float` for float and `identity` for characters. Default is `float`. 
