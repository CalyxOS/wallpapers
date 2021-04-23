# CalyxOS Wallpapers

Public Domain and Creative Commons licensed wallpapers.

This is intended for use with the Blossom Wallpaper app, included with CalyxOS, but obviously you can use this anywhere license compatible.

See https://calyxos.gitlab.io/wallpapers/

Usage:

1. edit sources.yml
2. rake
3. open ./public/index.html

## License

All images, and this repository, are licensed under CC BY-SA 3.0, unless otherwise noted.

https://creativecommons.org/licenses/by-sa/3.0/

## JSON API

Running `rake` generates two static galleries for the images: one is rendered as HTML and other as JSON.

The JSON roughly follows a very limited subset of the [Unsplash API](https://unsplash.com/documentation), except it is static and so there is no support for pagination or dynamic queries.

Supported actions:

* `GET /photos`: returns a JSON index of all the photos.

* `GET /photos/:id`: returns JSON for a single photo.

## Currently Used Sources

* https://commons.wikimedia.org/wiki/Main_Page
* https://bazaar.launchpad.net/~ubuntu-art-pkg/ubuntu-wallpapers/ubuntu/files

## Potential Sources

CC and public domain sources:

* https://commons.wikimedia.org/wiki/Main_Page
* https://commons.wikimedia.org/wiki/Commons:Picture_of_the_Year
* https://bazaar.launchpad.net/~ubuntu-art-pkg/ubuntu-wallpapers/ubuntu/files
* https://en.wikipedia.org/wiki/Wikipedia:Public_domain_image_resources
* https://www.loc.gov/free-to-use/

Other OS wallpaper collections:

* https://github.com/pop-os/wallpapers
* https://github.com/Antergos/wallpapers-extra
* https://github.com/elementary/wallpapers

## Wallpaper Android Apps

* https://github.com/romannurik/muzei
* https://github.com/danimahardhika/wallpaperboard -- heavily customizable, connect to anything using JSON API.
* https://github.com/GreyLabsDev/PexWalls -- client for pexels.com
* https://github.com/biniamHaddish/WallPack -- client for unsplash.com
* https://github.com/JuniperPhoton/MyerSplash.Android -- client for unsplash.com
* https://github.com/hoc081098/wallpaper-flutter
* https://github.com/Hash-Studios/Prism

## Device Screen Sizes

| Pixel 2     | 1080 x 1920 | 9:16   |
| Pixel 2 XL  | 1440 x 2880 | 9:18   |
| Pixel 3     | 1080 x 2160 | 9:18   |
| Pixel 3 XL  | 1440 x 2960 | 9:18.5 |
| Pixel 3a    | 1080 x 2220 | 9:18.5 |
| Pixel 3a XL | 1080 x 2160 | 9:18   |
| Pixel 4     | 1080 x 2280 | 9:19   |
| Pixel 4 XL  | 1440 x 3040 | 9:19   |
| Pixel 4a    | 1080 x 2340 | 9:19.5 |
| Mi A2       | 1080 x 2160 | 9:18   |

The maximum screen height is 3040px and the maximum width is 1440px.

## See also

* https://gitlab.com/calyxos/platform_packages_apps_Backgrounds
* https://gitlab.com/CalyxOS/platform_packages_apps_Backgrounds/-/tree/android11-qpr1/res/drawable-nodpi

## TODO

* The JSON files do not currently include the height or width of the image.
