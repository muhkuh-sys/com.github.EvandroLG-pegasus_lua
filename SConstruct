# -*- coding: utf-8 -*-
#-------------------------------------------------------------------------#
#   Copyright (C) 2019 by Christoph Thelen                                #
#   doc_bacardi@users.sourceforge.net                                     #
#                                                                         #
#   This program is free software; you can redistribute it and/or modify  #
#   it under the terms of the GNU General Public License as published by  #
#   the Free Software Foundation; either version 2 of the License, or     #
#   (at your option) any later version.                                   #
#                                                                         #
#   This program is distributed in the hope that it will be useful,       #
#   but WITHOUT ANY WARRANTY; without even the implied warranty of        #
#   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the         #
#   GNU General Public License for more details.                          #
#                                                                         #
#   You should have received a copy of the GNU General Public License     #
#   along with this program; if not, write to the                         #
#   Free Software Foundation, Inc.,                                       #
#   59 Temple Place - Suite 330, Boston, MA  02111-1307, USA.             #
#-------------------------------------------------------------------------#


#----------------------------------------------------------------------------
#
# Set up the Muhkuh Build System.
#

SConscript('mbs/SConscript')
Import('atEnv')

import os.path
import patch
import tarfile


#----------------------------------------------------------------------------
#
# Depack the source archive.
#
tSrcArchive = tarfile.open('pegasus.lua-0.9.6.tar.gz', 'r')
tSrcArchive.extractall('targets/depack')
tSrcArchive.close()
strDepackPath = 'targets/depack/pegasus.lua-0.9.6/'

#----------------------------------------------------------------------------
#
# Apply patches.
#
strPatchFolder = 'patches'
uiStrip = 1

# Collect all ".diff" files from the patch folder.
astrPatches = []
for strDirname, astrDirnames, astrFilenames in os.walk(strPatchFolder):
    for strFilename in astrFilenames:
        strDummy, strExt = os.path.splitext(strFilename)
        if strExt == '.diff':
            strAbsFilename = os.path.join(strDirname, strFilename)
            astrPatches.append(strAbsFilename)

# Sort the patches alphabetically.
astrSortedPatches = sorted(astrPatches)
for strPatch in astrSortedPatches:
    print('Apply patch "%s"...' % strPatch)

    # Apply the patches.
    tPatch = patch.fromfile(strPatch)
    tPatch.diffstat()
    tResult = tPatch.apply(uiStrip, root=strDepackPath)
    if tResult is not True:
        raise Exception('Failed to apply patch "%s"!' % strPatch)

#----------------------------------------------------------------------------
#
# Build the artifacts.
#

strGroup = 'com.github.EvandroLG'
strModule = 'pegasus_lua'

# Split the group by dots.
aGroup = strGroup.split('.')
# Build the path for all artifacts.
strModulePath = 'targets/repository/%s/%s/%s' % ('/'.join(aGroup), strModule, PROJECT_VERSION)

# Set the name of the artifact.
strArtifact = 'pegasus.lua'

tArcList = atEnv.DEFAULT.ArchiveList('zip')

tArcList.AddFiles('',
                    'installer/install.lua')

tArcList.AddFiles('lua/pegasus',
                  os.path.join(strDepackPath, 'src', 'pegasus', 'compress.lua'),
                  os.path.join(strDepackPath, 'src', 'pegasus', 'handler.lua'),
                  os.path.join(strDepackPath, 'src', 'pegasus', 'init.lua'),
                  os.path.join(strDepackPath, 'src', 'pegasus', 'request.lua'),
                  os.path.join(strDepackPath, 'src', 'pegasus', 'response.lua'))


tArtifact = atEnv.DEFAULT.Archive(os.path.join(strModulePath, '%s-%s.zip' % (strArtifact, PROJECT_VERSION)), None, ARCHIVE_CONTENTS = tArcList)
tArtifactHash = atEnv.DEFAULT.Hash('%s.hash' % tArtifact[0].get_path(), tArtifact[0].get_path(), HASH_ALGORITHM='md5,sha1,sha224,sha256,sha384,sha512', HASH_TEMPLATE='${ID_UC}:${HASH}\n')
tConfiguration = atEnv.DEFAULT.Version(os.path.join(strModulePath, '%s-%s.xml' % (strArtifact, PROJECT_VERSION)), 'installer/%s.xml' % strModule)
tConfigurationHash = atEnv.DEFAULT.Hash('%s.hash' % tConfiguration[0].get_path(), tConfiguration[0].get_path(), HASH_ALGORITHM='md5,sha1,sha224,sha256,sha384,sha512', HASH_TEMPLATE='${ID_UC}:${HASH}\n')
tArtifactPom = atEnv.DEFAULT.ArtifactVersion(os.path.join(strModulePath, '%s-%s.pom' % (strArtifact, PROJECT_VERSION)), 'installer/pom.xml')
