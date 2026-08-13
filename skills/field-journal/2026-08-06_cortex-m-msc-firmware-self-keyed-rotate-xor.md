

- auth_basis: own_system
- asset_types: [firmware_container, cortex_m_application, usb_msc_updater]


- lead_role: lead
- specialists: [cre, cce, doc]


|---|---|---|---|


- path_type: solve/callflow


|---|---|---|---|


```text
block_size = 0x400
mask = packed_block[0]
rotation(i) = (3 * i) mod 7
plain[i] = ROR8(packed[i], rotation(i)) XOR mask
```


- OS: Windows
