# telegram-desktop-stealth

Arch Linux package for Telegram Desktop with outbound activity-status
notifications removed.

## What it changes

The patch turns `Api::SendProgressManager::send()` into a no-op. That is the
single place where the client sends `messages.setTyping` actions, so the
following are no longer broadcast to the peer or group:

- typing
- recording audio, video or round video
- uploading files, photos, videos or voice messages
- choosing a sticker, location or contact
- playing a game
- speaking in a group call

Message content, file transfers, local UI behaviour and everything received
from other users are unchanged.

## What it does not change

- **Read receipts.** Marking a chat read (`messages.readHistory`) is the same
  request that clears your server-side unread state. Suppressing it hides the
  double check marks but breaks unread synchronisation across sessions, so it
  is intentionally left alone.
- **Online / last seen.** `account.updateStatus` is untouched.
- **Joining a group call.** Only the "speaking" action is suppressed; joining
  a call still puts you in the participant list.

## Version policy

The packaging is based on the version published in the Arch Linux `extra`
repository, not on the packaging repository's `main` branch:

- current base: `telegram-desktop 7.4.4-2`
- tag: `7.4.4-2` (`35e8bec68d6b`)

Arch bumps `pkgrel` without changing `pkgver` when it rebuilds a package
against a new library ABI; `7.4.4-2` is such a rebuild for libjxl 0.13.
Follow those rebuilds as well as new upstream versions.

This package pins the libjxl ABI with `libjxl.so=0.13-64` and
`libjxl_threads.so=0.13-64`. When libjxl bumps its soname, pacman reports an
unsatisfied dependency instead of letting the upgrade break Telegram at
runtime; rebuild this package against the new soname then.

## Build

GitHub Actions builds the package in an `archlinux:base-devel` container. The
resulting `*.pkg.tar.zst` is uploaded as a workflow artifact. No build is run
on the local machine.

LTO is disabled in the PKGBUILD because the final LTO link exceeds the memory
available on a GitHub-hosted runner. This affects optimisation only, not
behaviour.

## Install

```bash
sudo pacman -U telegram-desktop-stealth-7.4.4-2-x86_64.pkg.tar.zst
```

The package declares `provides=('telegram-desktop')` and
`conflicts=('telegram-desktop')`, so pacman will replace the official package.
Configuration and login data stay in `~/.local/share/TelegramDesktop`.

## License

The patch and packaging follow the upstream Telegram Desktop license
(`GPL-3.0-or-later WITH OpenSSL-exception`). The original PKGBUILD is from the
Arch Linux packaging repository.
