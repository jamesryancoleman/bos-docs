# Usage Concepts

Some key concepts of openBOS are:
- control points are uniformly accessible using their unique uris.
- control points uris should be queried from the system model, not memorized or hardcoded. 

# Examples
When possible, narrow down the point you want to access by its brick schema class, parent device/equipment class, and/or the space its located in.

``` python
>>> import bospy as bos
>>> space_temps = bos.query_points(
        kind='brick:Zone_Air_Temperature_Sensor',
        parent_kind='brick:Thermostat',
        space='1 - Ground Floor',
        )


```