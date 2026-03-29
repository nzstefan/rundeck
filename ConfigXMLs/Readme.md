## XML File Naming Rules

These XML files are used both by automation scripts and by staff performing manual checks. Because of that, the file name itself is part of the lookup process and must follow a consistent pattern.

### Purpose

The XML file name must clearly identify:

- the product
- the profile number
- optionally, human-readable notes

The automation uses the file name to determine which XML belongs to which product/profile.

### Required Naming Format

```text
<PRODUCT_NAME>_memory_check_Profile<NUMBER>.xml
```

Example:

```text
TICT_V2_memory_check_Profile2.xml
USRID_16_memory_check_Profile0.xml
TIC20_V4_WC_memory_check_Profile5.xml
```

### Rules

#### 1. Always include the profile number

Every XML file must include an explicit profile number.

Use:

```text
Profile0
Profile1
Profile2
...
```

Do not rely on filenames without a profile number, or just "logtag" to mean "default" or "profile 0".

Good:

```text
TIC20_V4_WC_memory_check_Profile0.xml
TICT_V2_memory_check_Profile2.xml
```

Avoid:

```text
TIC20_V4_WC_memory_check_LogTag.xml
TICT_V2_memory_check.xml
```

#### 2. Use unique names only

Every XML file must result in a unique combination of product id and profile. You must not create profiles that only differ in the comments.

Use:

```text
productID_memory_check_Profile0.xml
productID_memory_check_Profile1.xml
...
```

You must not rely on filename comments as the only means to distinguish profile files.

Good:

```text
TIC20_V4_WC_memory_check_Profile0_config_for_SI.xml
TIC20_V4_WC_memory_check_Profile1_config_for_ROW.xml
```

Avoid:

```text
TIC20_V4_WC_memory_check_Profile0_config_for_SI.xml
TIC20_V4_WC_memory_check_Profile0_config_for_ROW.xml
```

The parser will at best fail, at worst approve a product that is not correct.

#### 3. Use underscores in the product name

If the product name normally contains spaces, replace the spaces with underscores in the filename.

Examples:

- `TICT V2` -> `TICT_V2`
- `USRID 16` -> `USRID_16`
- `TIC20 V4 WC` -> `TIC20_V4_WC`

This keeps filenames consistent and easier to parse.

#### 4. Keep the product name at the start of the file

The filename must begin with the product name.

Good:

```text
TICT_V2_memory_check_Profile2.xml
```

Avoid:

```text
Profile2_TICT_V2_memory_check.xml
memory_check_TICT_V2_Profile2.xml
```

#### 5. Use `memory_check` exactly

The middle part of the filename must always be:

```text
_memory_check_
```

Good:

```text
TICT_V2_memory_check_Profile2.xml
```

Avoid:

```text
TICT_V2_memcheck_Profile2.xml
TICT_V2_memorycheck_Profile2.xml
TICT_V2_check_Profile2.xml
```

#### 6. Optional human-readable notes may be added after the profile

Extra notes for staff are allowed, as long as they come after the profile number. These notes are ignored by the profile matching logic.


Good:
```text
TICT_V2_memory_check_Profile2_LogTag.xml
TICT_V2_memory_check_Profile2_Manual_with_hibernation_for_CiK.xml
TIC20_V4_WC_memory_check_Profile0_Default.xml
```

Avoid:
```text
TICT_V2_memory_check_CiK_Profile2.xml
```


#### 7. Profile formats that are accepted

Automation currently accepts these profile styles:

- `Profile2`
- `Profile_2`
- `Profile-2`
- `Profile 2`

However, for consistency, the preferred format is:

```text
Profile2
```

Recommended:

```text
TICT_V2_memory_check_Profile2.xml
```

Not recommended, but may still work:

```text
TICT_V2_memory_check_Profile_2.xml
TICT_V2_memory_check_Profile-2.xml
```

#### 8. File extension must be `.xml`

Good:

```text
TICT_V2_memory_check_Profile2.xml
```

Avoid:

```text
TICT_V2_memory_check_Profile2.XML
TICT_V2_memory_check_Profile2.txt
```

### Recommended Standard

The recommended final format is:

```text
<PRODUCT_NAME_WITH_UNDERSCORES>_memory_check_Profile<NUMBER>[_OPTIONAL_NOTE].xml
```

Examples:

```text
TICT_V2_memory_check_Profile2.xml
TICT_V2_memory_check_Profile2_LogTag.xml
TIC20_V4_WC_memory_check_Profile0_Default.xml
USRID_16_memory_check_Profile5.xml
```



## XML Memory Check Profile Rules

 This part of the document describes the intended structure and validation rules for the memory-check XML files


### Purpose of the file
This XML is primarily a memory validation profile.
Each `<memory_check>` block defines:
- a field name
- a memory address
- a byte length
- an endian interpretation
- and either:
- one exact expected value, or
- an allowed min/max range

At the file level, the root metadata identifies which logger model and firmware version the rule set applies to.

### Intended overall structure

```xml
<?xml version="1.0" encoding="utf-8"?>
<basemem_check_info>
    <format_ver>00</format_ver>
    <logger_model>MODEL NAME</logger_model>
    <firmware_version>FIRMWARE STRING</firmware_version>
    <memory_check>
        <name>FIELD_NAME</name>
        <mem_address_hex>00</mem_address_hex>
        <length>1</length>
        <big_endian>yes</big_endian>
        <range>no</range>
        <value>FE</value>
    </memory_check>
    <memory_check>
        <name>FIELD_NAME_2</name>
        <mem_address_hex>0100</mem_address_hex>
        <length>2</length>
        <big_endian>yes</big_endian>
        <range>yes</range>
        <value_min>0001</value_min>
        <value_max>00FF</value_max>
    </memory_check>
</basemem_check_info>
```
 
### Structural rules

#### 1. XML declaration

- Must appear first in the file.
- Recommended form:
<?xml version="1.0" encoding="utf-8"?>

#### 2. Root element

- The file must contain exactly one root element:
- `<basemem_check_info>`

#### 3. Root metadata order

The following elements should appear once each, directly under `<basemem_check_info>`, before any `<memory_check>` blocks:
1. `<format_ver>`
2. `<logger_model>`
3. `<firmware_version>`

#### 4. Repeating payload block

- Each memory rule must be wrapped in its own `<memory_check>` element.
- A `<memory_check>` block must not contain another `<memory_check>` block.
- Every logical check should be self-contained.

#### 5. Child element order inside `<memory_check>`
Recommended order:
1. `<name>`
2. `<mem_address_hex>`
3. `<length>`
4. `<big_endian>`
5. `<range>`
	1. exact-value form:
		- `<value>`
	2. range-value form:
		- `<value_min>`
		- `<value_max>`
  
### Semantic rules

#### Exact-match check
If:
<range>no</range>
then:
- `<value>` is required
- `<value_min>` must not be present
- `<value_max>` must not be present

Example:
```xml
<memory_check>
    <name>QC PASSCODE</name>
    <mem_address_hex>01</mem_address_hex>
    <length>1</length>
    <big_endian>yes</big_endian>
    <range>no</range>
    <value>3F</value>
</memory_check>
```

#### Range check
If:
<range>yes</range>
then:
- `<value_min>` is required
- `<value_max>` is required
- `<value>` must not be present

Example:
```xml
<memory_check>
    <name>LOGINTERVAL</name>
    <mem_address_hex>C8</mem_address_hex>
    <length>2</length>
    <big_endian>yes</big_endian>
    <range>yes</range>
    <value_min>0001</value_min>
    <value_max>FFFF</value_max>
</memory_check>
```
  

### Element rules

| Element | Parent | Required | Allowed values / format | Meaning | Placement rule | Notes |
|---|---|---:|---|---|---|---|
| `<?xml ...?>` | file | Yes | standard XML declaration | Declares XML version and encoding | First line in file | Should precede all content |
| `<basemem_check_info>` | file | Yes | single root element | Root container for the whole profile | One time only | Everything else must be inside it |
| `<format_ver>` | `basemem_check_info` | Yes | short text, usually numeric version string | Defines XML format version | Before any `memory_check` blocks | Example: `00` |
| `<logger_model>` | `basemem_check_info` | Yes | text | Identifies target logger model | Before any `memory_check` blocks | Example: `USRIC-8 V4` |
| `<firmware_version>` | `basemem_check_info` | Yes | text | Identifies applicable firmware version | Before any `memory_check` blocks | Example: `6.69 B61` |
| `<memory_check>` | `basemem_check_info` | Yes, repeating | container element | Defines one memory validation rule | After root metadata | Repeat once per field |
| `<name>` | `memory_check` | Yes | text | Human-readable field name | First child recommended | Example: `QC PASSCODE` |
| `<mem_address_hex>` | `memory_check` | Yes | hex string, no `0x` prefix | Start memory address of the field | After `name` | Example: `00`, `0100`, `0288` |
| `<length>` | `memory_check` | Yes | integer `1` to `4` | Field width in bytes | After `mem_address_hex` | Must match encoded value width |
| `<big_endian>` | `memory_check` | Yes | `yes` or `no` | Controls byte interpretation for multibyte values | After `length` | `yes` = MSB first |
| `<range>` | `memory_check` | Yes | `yes` or `no` | Indicates exact-match or range-match validation | After `big_endian` | Governs which value elements are legal |
| `<value>` | `memory_check` | Conditional | hex string | Exact expected value | Only when `range=no` | Must not appear when `range=yes` |
| `<value_min>` | `memory_check` | Conditional | hex string | Minimum acceptable value | Only when `range=yes` | Must not appear with `<value>` |
| `<value_max>` | `memory_check` | Conditional | hex string | Maximum acceptable value | Only when `range=yes` | Must not appear with `<value>` |

  

---

  

### Validation rules

  

#### Address format

- `mem_address_hex` must be hexadecimal text.

- It should not include `0x`.

- Uppercase hex is recommended for consistency.

  

Valid examples:

```xml
<mem_address_hex>00</mem_address_hex>
<mem_address_hex>20</mem_address_hex>
<mem_address_hex>0100</mem_address_hex>
<mem_address_hex>0288</mem_address_hex>
```
 

#### Length rule

- `length` defines the number of bytes to read from memory.
- Allowed values are intended to be 1, 2, 3, or 4.

#### Value width rule

The hex string width should match the byte length.
Examples:
- `length=1` -> 2 hex digits
- `length=2` -> 4 hex digits
- `length=3` -> 6 hex digits
- `length=4` -> 8 hex digits

Examples:

```
<length>1</length>
<value>FE</value>

<length>2</length>
<value>0011</value>
```
  
#### Endianness rule

- `big_endian=yes` means multibyte values are interpreted most-significant byte first.

- `big_endian=no` means multibyte values are interpreted least-significant byte first.

- For single-byte values, endianness is logically irrelevant but may still be present for schema consistency.

#### Range exclusivity rule

Exactly one of these modes must be used:

##### Exact mode

```xml
<range>no</range>
<value>...</value>
```
##### Range mode

```xml
<range>yes</range>
<value_min>...</value_min>
<value_max>...</value_max>
```
  
Do not mix them in one block.

### Logical meaning of a memory check

Each `<memory_check>` block means:

1. Read `length` bytes starting at `mem_address_hex`
2. Interpret them using the specified endian rule
3. Compare the result:
- to `<value>` if `range=no`
- or between `<value_min>` and `<value_max>` if `range=yes`

### Practical grouping observed in the example file

These files may be loosely grouped by address range.

#### 00 to 7F

Typically device identity, USB, flash, and hardware config.

Examples of names seen in this region:

- `CUSTOMISATION STATE`
- `QC PASSCODE`
- `PRODUCT_ID`
- `USB_VID`
- `USB_PID`
- `SPIFLASH_SIZE`
- `HARDWARE_CONFIG`

#### 80 to BF

Typically PDF, display, and branding options.

Examples:

- `PDF_TIME_ZONE`
- `PDF_OPTIONS`
- `PDF_CHART_OPTIONS`
- `PDFPASSWORD`
- `PDF_BRANDNAME`
- `PDF_LOGO_*`

#### C0 to CF

Typically logging configuration and logger control/status.

Examples:
- `TOTALLOGS`
- `LOGBUFFERSIZE`
- `STARTUPDELAY`
- `LOGINTERVAL`
- `LCONTROL`
- `LSTATUS`

#### F0 onward

Typically display timing and control.

Examples:

- `DISPLAY_ON_TIME`
- `DISPLAY_SCROLL_TIME`
- `DISPLAY_CONTROL`

#### 0100 onward

Typically alarm settings and flags.

Examples:
- `MASTER_ALM_CTRL`
- `A1CONTROL`
- `A1DELAY`
- `A1THRESVAL`
- `A2CONTROL`
- `DATA_FLAGS`

#### 0280 onward

Typically table or calibration style information.

Examples:

- `TABLEID`
- `NUMENTRIES`
- `TOPVAL`
- `BOTTOMVAL`
- `STEP`


### Typical Problems found in malformed files

  

#### 1. Invalid opening comments

The opening comment block is not legal XML because nested comment syntax is used.
This should be removed entirely.

#### 2. Missing `<memory_check>` wrappers

At least some visible entries appear to contain `<name>` and related children without an opening `<memory_check>` wrapper.

That means every field definition should be checked to ensure it is wrapped like this:

```xml
<memory_check>

...

</memory_check>
```
  

#### 3. Suspicious duplicated field name

For example, one field name appears duplicated:

- `A4THRESVAL`

This may be intentional, but it should be reviewed.

### Recommended cleaned schema
Use this as the normalized rulebook for future files:

```
<?xml version="1.0" encoding="utf-8"?>
<basemem_check_info>
    <format_ver>00</format_ver>
    <logger_model>LOGGER MODEL</logger_model>
    <firmware_version>FIRMWARE VERSION</firmware_version>
    
    <memory_check>
        <name>FIELD_NAME</name>
        <mem_address_hex>HEX_ADDRESS</mem_address_hex>
        <length>1</length>
        <big_endian>yes</big_endian>
        <range>no</range>
        <value>HEX_VALUE</value>
    </memory_check>
    
    <memory_check>
        <name>FIELD_NAME</name>
        <mem_address_hex>HEX_ADDRESS</mem_address_hex>
        <length>2</length>
        <big_endian>yes</big_endian>
        <range>yes</range>
        <value_min>MIN_HEX_VALUE</value_min>
        <value_max>MAX_HEX_VALUE</value_max>
    </memory_check>
</basemem_check_info>
```



# Summary

## File naming

#### Required

- product name at the beginning
- underscores instead of spaces
- `_memory_check_`
- explicit `Profile<number>`
- `.xml` extension

#### Allowed

- optional note after the profile number

#### Avoid

- missing profile number
- inconsistent separators
- product name not at the start
- alternate wording instead of `memory_check`

### Reason for These Rules

These files are selected by filename during automated processing. If the name is inconsistent, the wrong XML may be used or no XML may be found at all.

A consistent naming standard also makes manual lookup easier for staff.  

These files are intended to describe memory validation rules for a specific logger model and firmware version.

  
## Composition
The core structure is:
- one XML declaration
- one `<basemem_check_info>` root
- three metadata fields near the top
- many repeated `<memory_check>` rule blocks
