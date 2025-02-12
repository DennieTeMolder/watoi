# Whatsapp Android To iOS Importer

Migrate Whatsapp chats history from Android to iOS.

## Caveats

* Media files and shared locations are not imported (got placeholders instead)
* Messages from contacts that changed ids (phone numbers) are not linked

## Prerequisites

* Mac with installed Xcode and iTunes
* Decrypted `msgstore.db` from Android
* Installed and activated Whatsapp on your iDevice
* `Whatsapp.ipa` of the same version (google will help)

## Step-by-step guide

* Check that Whatsapp is activated on iDevice. You should see the list of *group* chats
  when you open the app. Most likely, there won't be any messages prior to moving to iOS.
  You can even send/receive a message or two to be sure that there is something to back up.
* If you're on Mac OS 10.14 or later, [enable full disk access for your terminal app](https://osxdaily.com/2018/10/09/fix-operation-not-permitted-terminal-error-macos/).
* Build the migration utility (I'll assume `~/Downloads` folder):

      cd ~/Downloads
      git clone https://github.com/residentsummer/watoi
      cd watoi
      xcodebuild -project watoi.xcodeproj -target watoi

* Create an unencrypted backup to local computer (not iCloud) with iTunes.
* Find out the ID of the latest backup and export it into an env var for later use

      scripts/bedit.sh list-backups
      # total 0
      # drwxr-xr-x@   266 user  staff    8512 Nov 19 22:25 3105fe2b1e4688d54920d5b7eff3a06a71fd5957  # <= this is the latest
      # drwxr-xr-x  20781 user  staff  664992 Aug 16 14:19 e38ebede76a7801807fe98684fd6d0b7fc3e64ba

      export BACKUP_ID="put ID of the backup here"
      # For example (do not copy-paste the line below, your ID will be different!):
      # export BACKUP_ID=3105fe2b1e4688d54920d5b7eff3a06a71fd5957

* Extract whatsapp's chat storage and backup important files

      export ORIGINALS="originals/$(date +%s)"
      mkdir -p $ORIGINALS
      scripts/bedit.sh extract-chats $BACKUP_ID $ORIGINALS/ChatStorage.sqlite
      scripts/bedit.sh extract-blob $BACKUP_ID Manifest.db $ORIGINALS/Manifest.db
      cp $ORIGINALS/ChatStorage.sqlite ./ChatStorage.sqlite

* Extract the contents of `Whatsapp.ipa` (we'll need CoreData description files):

      cd ~/Downloads/watoi
      unzip ~/Downloads/WhatsApp_Messenger_x.y.z.ipa -d app

* Run the migration:

      build/Release/watoi <path-to-msgstore.db> ./ChatStorage.sqlite app/Payload/WhatsApp.app/Frameworks/Core.framework/WhatsAppChat.momd

* Replace database file inside the backup with the updated one:

      scripts/bedit.sh replace-chats $BACKUP_ID ./ChatStorage.sqlite

* Restore the backup with iTunes

## Troubleshooting

[![Gitter](https://badges.gitter.im/gitterHQ/gitter.svg)](https://gitter.im/residentsummer_watoi/Lobby?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)
