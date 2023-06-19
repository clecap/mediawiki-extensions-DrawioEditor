# Patches in this repository

This repository contains a few patches of the MediaWiki extension DrawEditor https://www.mediawiki.org/wiki/Extension:DrawioEditor


## 1. Fix path setting in ext.drawioeditor.js:

The path setting mechanism in `$wgDrawioEditorBackendUrl` in the original is constructed in a way to conform
the requirements of the Dockerfile described in https://github.com/jgraph/docker-drawio. This Dockerfile assumes the
existence of a Tomcat server which redirects a directory URL to an index.html file.

In our installation we do not use this Dockerfile but employ a more light-weight installation. Therefore
we remove the slash before the `?embed` in `src: this.baseUrl + '?embed=1&proto=json&spin=1&analytics=0&picker=0&lang=' + this.language + localAttr`
in line 52 of `ext.draweditor.js`. This allows us to specify a file as endpoint in `$wgDrawioEditorBackendUrl`.

In my test-installation this amounts to `$wgDrawioEditorBackendUrl="http://localhost:8080/experiments/drawio-dev/src/main/webapp/index.html";`

This is fixed by my commit https://github.com/wikimedia/mediawiki-extensions-DrawioEditor/commit/037e684fc9968244b60e9183249ae8b97c59c93f

## 2. Do Due Diligence

Given the problems described in https://www.mediawiki.org/wiki/Extension_talk:DrawioEditor, it is recommendable to check whether you have done the following:
* Allow upload of files in the wiki configuration
* Allow upload of filetype `svg` in wiki configuration
* Check if the file indeed shows up in image directory

## 3. Patch bug in ext.drawioeditor.js

In file `ext.drawioeditor.js` the error reporting is broken in function `DrawioEditor.prototype.uploadToWiki`. Therefore,
the extension does not properly report errors and exceptions caused by the MediaWiki system.

This bug needs adding an additional case and error output in line 218 of `ext.draweditor.js`.

This is fixed by my commit https://github.com/wikimedia/mediawiki-extensions-DrawioEditor/commit/037e684fc9968244b60e9183249ae8b97c59c93f


## 4. Fix includes/libs/ParamValidator/Util/UploadedFile.php

When the error reporting is fixed as above, one indeed gets to see some errors which still exist MediaWiki in file `includes/libs/ParamValidator/Util/UploadedFile.php`.

`Declaration of Wikimedia\ParamValidator\Util\UploadedFile::getStream() must be compatible with Psr\Http\Message\UploadedFileInterface::getStream(): 
Psr\Http\Message\StreamInterface in <b>/var/www/html/wiki-dir/includes/libs/ParamValidator/Util/UploadedFile.php</b> on line <b>86</b><br />`

### Patches to be done to MediaWiki core file `includes/libs/ParamValidator/Util/UploadedFile.php`
* The function `getStream()` in line 86 lacks the specification StreamInterface:  `public function getStream(): StreamInterface`
* The function `moveTo($targetPath)`in line 96 lacks the specification `moveTo(string $targetPath): void`
* The functtion `getSize()` in line 121 lacks the specification `?int`
* The function `getError()` in line 125 lacks the specification `int`
* The function `getClientFilename()`in line 129 lacks the specification `?string`
* The function `getClientMediaType()`in line 134 lacks the specification `?string`
* Moreover PHP requires path access to the proper file describing `StreamInterface`, so we must add 
`use Psr\Http\Message\StreamInterface;`to the header in this file.

The correct file is supplied here as PATCH-UploadedFile.php (for MediaWiki 1.40).

See also: https://www.mediawiki.org/w/index.php?title=Topic:Xkagz9y10nf2dq26&topic_showPostId=xkajubqfr0vfw821#flow-post-xkajubqfr0vfw821

## 5. Further Bug:

We need to do a reload after we return from the editor to the Wikipage.

This has been fixed by an add on to `DrawioEditor.prototype.exit` in ext.drawioeditor.js

## 6. Extension.json

* Changed `config.DrawioEditorImageType` to "svg"
* Changed `DrawioEditorBackendUrl.value`to "" to prevent any use outside of the configuration which is done in DanteSettings.php


## 7. Type bug

In file `src/Hook/ApprovedRevsSetStableFile.php` we remove the type specification ùser`at variable `$user`
in `public function onDrawioGetFile( File &$file, &$latestIsStable, User $user, bool &$isNotApproved, &$displayFile )`
as this produces errors in `danteEndpoint.php` previews.


# Differences in Usage

<drawio filenae="some-name" /> also works 
