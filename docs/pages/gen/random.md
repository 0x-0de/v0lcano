# `random`

Contains random number generators and noise functions.

## NoiseProperties (`struct`)

Properties used for generating value noise.

### Fields

`octaves: u8` - Number of layers of detail to generate.

`focus: f64` - Controls how much each additional layer of noise scales down.

`persistance: f64` - Controls how much each additional layer should add to the noise.

## Public Functions

**`random_float(comptime T_float: type, comptime T_int: type, comptime seed_count: u8, seeds: [seed_count]u256, min: T_float, max: T_float) T_float`**

Returns a random float between two boundary values. Type T must be a float of no greater than 256 bits.

**`random_int(comptime T: type, comptime seed_count: u8, seeds: [seed_count]u256) T`**

Returns a random integer value generated using the seed values. Type T must be an int of no greater than 256 bits.

**`value_noise_1d(seed: u256, x: f64, properties: NoiseProperties) f64`**

Returns a value noise value, between -1 and 1 (inclusive). Generated along one axis.

**`value_noise_2d(seed: u256, x: f64, y: f64, properties: NoiseProperties) f64`**

Returns a value noise value, between -1 and 1 (inclusive). Generated along two axes.

## Private Fields

```
const permutations: [256]u8 = .{
    144, 219, 232, 110, 171, 202, 50,  242, 55,  148, 87,  6,   12,  152, 143, 21,
    27,  195, 180, 249, 79,  139, 134, 98,  103, 37,  41,  125, 213, 48,  159, 160,
    10,  46,  128, 157, 236, 181, 124, 168, 251, 184, 19,  149, 54,  189, 61,  127,
    138, 105, 16,  145, 76,  210, 113, 169, 94,  252, 255, 158, 246, 156, 175, 65,
    203, 64,  225, 18,  17,  142, 30,  226, 71,  137, 118, 13,  216, 59,  126, 116,
    194, 186, 174, 45,  234, 97,  220, 217, 206, 235, 182, 170, 153, 106, 4,   114,
    166, 237, 108, 7,   36,  26,  177, 223, 243, 163, 164, 141, 66,  212, 28,  197,
    86,  240, 63,  132, 211, 227, 57,  104, 0,   72,  39,  58,  123, 221, 2,   228,
    62,  214, 229, 218, 99,  101, 112, 11,  207, 75,  32,  47,  196, 68,  42,  34,
    92,  85,  190, 43,  135, 154, 247, 173, 215, 80,  15,  90,  131, 84,  31,  51,
    187, 230, 172, 24,  40,  198, 115, 222, 8,   14,  60,  29,  23,  53,  204, 73,
    25,  9,   136, 117, 130, 241, 83,  74,  248, 5,   70,  208, 201, 82,  38,  147,
    56,  81,  183, 52,  121, 185, 1,   253, 91,  49,  96,  67,  111, 233, 188, 179,
    151, 119, 109, 95,  254, 167, 77,  176, 122, 44,  93,  238, 35,  245, 165, 205,
    200, 146, 239, 161, 100, 88,  209, 244, 120, 3,   231, 102, 193, 78,  155, 162,
    140, 20,  191, 199, 150, 224, 133, 178, 250, 89,  33,  129, 22,  69,  107, 192
};
```

List of permutations used by the random number generators.