---
description: Index into a slotted array and assign a sub-object to another slot
---

# slot\_subarray

### Behaviour

* Look up the array in slot `parent_slot`
* Retrieve the sub-object at the index specified in `array_id`
* Place sub-object into the slot `new_slot` or the next available slot if `new_slot` is 0.
* Return the new slot number.

### Definition

C

```c
int64_t slot_subarray (
    uint32_t parent_slot,
  	uint32_t array_id,
  	uint32_t new_slot
);
```

### Example

C

```c
int64_t subslot = 0;
subslot =
    slot_subarray(slot_no, i, (uint32_t)subslot);
```

### Parameters

| Name         | Type      | Description                                                                                                                                                                                                                                                                                                                                                                                                |
| ------------ | --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| parent\_slot | uint32\_t | Slot the parent array is in                                                                                                                                                                                                                                                                                                                                                                                |
| array\_id    | uint32\_t | <p>The <code>sf</code> code of the field you are searching for.<br><br>To compute this manually take the serialized <code>type</code> and shift it into the 16 highest bits of uint32_t, then take the <code>field</code> and place it in the 16 lowest bits.<br><br>For example:<br><code>sfEmitNonce</code> has <code>type</code> 5 and <code>field</code> 11 thus its value is <code>0x050BU</code></p> |
| new\_slot    | uint32\_t | New slot number to place the object from the selected array index into. If null, choose the next available slot. _May be null._                                                                                                                                                                                                                                                                            |

### Return Code

| Type     | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| int64\_t | <p>The slot number of the newly allocated object<br><br>If negative, an error:<br><br><code>DOESNT_EXIST</code><br>- The specified <code>array_id</code> doesn't exist in the array pointed to by <code>parent_slot</code><br><br><code>NO_FREE_SLOTS</code><br>- The API would require a new slot to be allocated but the Hook is already at the maximum number of slots.<br><br><code>NOT_AN_ARRAY</code><br>- The specified <code>parent_slot</code> does not contain an STArray.</p> |