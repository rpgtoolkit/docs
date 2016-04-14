## File Formats

The editor and engine components use a variety of specialized file formats and schemas to
facilitate game development and to support various features common in role-playing games.

### Legacy

The engine has some backwards compatibility with older (legacy) versions. The legacy file
formats are binary formats that use [little-endian](https://en.wikipedia.org/wiki/Endianness)
byte order.

- [Animation](#animation)
- [Background](#background)
- [Board](#board)
- [Enemy](#)
- [Item](#)
- [Project (Main File)](#)
- [Player](#)
- [Special Move](#)
- [Status Effect](#)
- [Tile](#)
- [Tile Animation](#)
- [Tile Bitmap](#)
- [Tileset](#)

#### Data Types

| Name          | Description                                               | Size |
| ------------- | --------------------------------------------------------- | ---- |
| u8            | unsigned 8-bit integer                                    | 1    |
| bool          | unsigned 8-bit boolean (false = 0, true &ne; 0)           | 1    |
| u16           | unsigned 16-bit little-endian integer                     | 2    |
| u32           | unsigned 32-bit little-endian integer                     | 4    |
| u64           | unsigned 64-bit little-endian integer                     | 8    |
| s16           | signed 16-bit little-endian integer                       | 2    |
| s32           | signed 32-bit little-endian integer                       | 4    |
| s64           | signed 64-bit little-endian integer                       | 8    |
| f32           | IEEE 754 single-precision floating-point number           | 4    |
| f64           | IEEE 754 double-precision floating-point number           | 8    |
| ascii         | null-terminated ASCII-encoded string                      | *    |
| rgb           | r8g8b8 packed color                                       | 4    |

#### Animation Frame

| Field              | Data Type | Description                                                    |
| ------------------ | --------- | -------------------------------------------------------------- |
| Path               | ascii     | Path to the frame image                                        |
| Transparent Color  | rgb       | Color to be replaced with transparency when drawn              |
| Sound              | ascii     | Path to sound to play when frame is drawn                      |

#### Animation

**Supported versions**: 2.3

| Field              | Data Type | Description                                                    |
| ------------------ | --------- | -------------------------------------------------------------- |
| Type               | ascii     | RPGTLKIT ANIM                                                  |
| Major Version      | u16       | Major version                                                  |
| Minor Version      | u16       | Minor version                                                  |
| Width              | u32       | Animation width (x bound)                                      |
| Height             | u32       | Animation height (y bound)                                     |
| Frame Count        | u32       | # of animation frames                                          |
| Frames             | Animation Frame [n] | Animation frames (n = **Frame Count**)               |
| Delay              | f32       | Delay (in seconds) between frames                              |

#### Background

**Supported versions**: 2.3

| Field              | Data Type | Description                                                    |
| ------------------ | --------- | -------------------------------------------------------------- |
| Type               | ascii     | RPGTLKIT BKG                                                   |
| Major Version      | u16       | Major version                                                  |
| Minor Version      | u16       | Minor version                                                  |
| Background Image   | ascii     | Path to background image to draw                               |
| Music              | ascii     | Path to audio file to play as background music                 |
| Selection Sound    | ascii     | Path to audio file to play when player moves through menu      |
| Choice Sound       | ascii     | Path to audio file to play when player chooses from menu       |
| Ready Sound        | ascii     | Path to audio file to play when player is ready                |
| Invalid Sound      | ascii     | Path to audio file to play when player makes an invalid choice |

#### Board

> This documentation is incomplete.

**Supported versions**: 2.4

| Field              | Data Type | Description                                                    |
| ------------------ | --------- | -------------------------------------------------------------- |
| Type               | ascii     | RPGTLKIT BOARD                                                 |
| Major Version      | u16       | Major version                                                  |
| Minor Version      | u16       | Minor version                                                  |
| Width              | u16       | Board width (in tiles)                                         |
| Height             | u16       | Board height (in tiles)                                        |
| Layer Count        | u16       | Board layer count                                              |
| Coordinate Type    | Coordinate Type | Coordinate type (e.g. ortho, isometric)                  |
| Tile LUT Size      | u16     | # of LUT entries                                                 |
| Tile LUT           | string [n] | See [Tile Lookup Table](#tile-lookup-table)                   |
| Tile Matrix        | u16 [n]    | See [Tile Matrix](#tile-matrix)                               |
| Shading Type       | u8         | Unused                                                        |

##### Tile Lookup Table

The tile lookup table (LUT) is a table of strings where each string is a path to a
single tile, an animated tile, or a path to a tileset with a tile index appended
to the file extension. Each tile in the board refers to an entry in the lookup table.

**Example LUT**

| Offset | Value            | Comments                |
| ------ | ---------------- | ----------------------- |
| 0      | grass.gph        | Single tile             |
| 1      | water.tan        | Animated tile           |
| 2      | terrain.tst12    | Tileset (tile # 12)     |

##### Tile Matrix

The tile matrix is a flattened 3-dimensional array of u16 integers referring to entries in the
tile lookup table. The tile matrix uses an optional and rudimentary run-length encoding (RLE)
compression scheme for repeating sequences of tiles.

**Psuedocode**

```
let size := width * height * depth
let offset := 0

while offset < size

  let index := read_u16()
  let count := 1

  if index < 0 then
    count = -index
    index = read_u16()
  end if

  for i = 0 to count - 1
    matrix[offset + i] := index
  end for

  offset := offset + count

end while
```


#### Coordinate Type

**Enumeration**  

Defines coordinate types used for directional movement, rendering, raycasting,
collision detection, and other mathematical transformations. A coordinate type defines the
render mode used for various game elements (e.g. boards).

| Field              | Data Type | Value | Description                                            |
| ------------------ | --------- | ----- | ------------------------------------------------------ |
| Orthogonal         | u32       | 1     | 2D orthogonal perspective                              |
| Isometric Stacked  | u32       | 2     | 2D stacked isometric perspective                       |
| Isometric Rotated  | u32       | 6     | 2D rotated isometric perspective                       |

#### Enemy

**Supported versions**: 2.1

| Field              | Data Type | Description                                                    |
| ------------------ | --------- | -------------------------------------------------------------- |
| Type               | ascii     | RPGTLKIT ENEMY                                                 |
| Major Version      | u16       | Major version                                                  |
| Minor Version      | u16       | Minor version                                                  |
| Name               | ascii     | Display Name                                                   |
| HP                 | u32       | Base HP                                                        |
| SMP                | u32       | Base SMP                                                       |
| FP                 | u32       | Base FP                                                        |
| DP                 | u32       | Base DP                                                        |
| Can Escape?        | bool      | Can player escape from the enemy?                              |
| Sneak Chance       | u16       | Chance of sneaking up on the enemy (1:n)                       |
| Enemy Sneak Chance | u16       | Chance of enemy sneaking up on the player (1:n)                |
| Special Move Count | u16       | # of special moves defined                                     |
| Special Moves      | ascii [n] | Paths of defined special moves usable in battle                |
| Weakness Count     | u16       | # of weaknesses                                                |
| Weaknesses         | ascii [n] | Paths to special moves the enemy is weak to                    |
| Strength Count     | u16       | # of strengths                                                 |
| Strengths          | ascii [n] | Paths to special moves the enemy is strong against             |
| AI Level           | AI Level  | See [AI Level](#ai-level)                                      |
| AI Program         | ascii     | Path to program to use for AI tactics                          |
| Use Tactics?       | bool      | Use a program for tactics instead of AI?                       |
| Experience Reward  | u32       | Experience awarded to player on victory condition              |
| Currency Reward    | u32       | Currency awarded to player on victory condition                |
| Victory Program    | ascii     | Path to program to execute on victory condition                |
| Escape Program     | ascii     | Path to program to execute on escape condition                 |
| Graphics Count     | u16       | # of enemy graphics                                            |
| Graphics           | ascii [n] | Paths to graphics for enemy stances                            |
| Custom Graphics Count | u16    | # of custom enemy graphics                                     |
| Custom Graphics    | ascii [n] | Pairs of (path, name) for custom enemy graphics                |

#### AI Level

Defines the level of artificial intelligence (AI) of an entity. The implementation of intelligence
is arbitrary and defined by the game programmer and/or a battle system plugin.

**Enumeration**

| Field              | Data Type | Value | Description                                            |
| ------------------ | --------- | ----- | ------------------------------------------------------ |
| Low                | u8        | 0     | Low intelligence                                       |
| Medium             | u8        | 1     | Medium intelligence                                    |
| High               | u8        | 3     | High intelligence                                      |
| Very High          | u8        | 4     | Very High intelligence                                 |

#### Item

**INCOMPLETE**  

**Supported Versions**: 2.7

| Field              | Data Type | Description                                                    |
| ------------------ | --------- | -------------------------------------------------------------- |
| Type               | ascii     | RPGTLKIT ITEM                                                  |
| Major Version      | u16       | Major version                                                  |
| Minor Version      | u16       | Minor version                                                  |
| Name               | ascii     | Item display name                                              |
| Description        | ascii     | Item description                                               |
| Can Equip?         | bool      | Can the item be equipped?                                      |
| Can Use In Menu?   | bool      | Can the item be used in the menu?                              |
| Can Use On Board?  | bool      | Can the item be used on a board?                               |
| Can Use In Battle? | bool      | Can the item be used in battle?                                |
| Used By            | u8        |                                                                |
| --- | | |
| Buy Price          | u32       | Price item can be bought for                                   |
| Sell Price         | u32       | Price item be sold for                                         |
| Is Key Item?       | bool      | Is the item a key / quest item?                                |
