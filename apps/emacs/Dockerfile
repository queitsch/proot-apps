# GNU Emacs as a PRoot App.
#
# Two builds of the same Emacs release live in one root:
#   /usr/bin/emacs      pure GTK (pgtk) build, native Wayland
#   /usr/libexec/emacs-x11/emacs  GTK3 build for X11
# The X11 binary keeps the file name emacs so its X11 class stays Emacs, which
# the desktop entry's StartupWMClass names.
# Alpine's emacs-pgtk and emacs-gtk3 packages conflict with each other, so the
# X11 binary and its dump file are taken from a second stage. Both packages
# ship byte identical Lisp, only the binary and its fingerprint named .pdmp
# differ. The entrypoint picks the build that matches the session.

ARG BASE_IMAGE=ghcr.io/linuxserver/proot-apps:alpine-324

# X11 (GTK3) build
FROM ${BASE_IMAGE} AS x11

RUN \
  echo "**** install emacs-gtk3 ****" && \
  apk add --no-cache \
    emacs-gtk3 && \
  echo "**** collect the X11 binary, its dump file and its libraries ****" && \
  mkdir -p /x11-out/usr/libexec/emacs-x11 && \
  cp /usr/bin/emacs /x11-out/usr/libexec/emacs-x11/emacs && \
  cd / && \
  tar -cf - usr/lib/emacs/*/*/emacs-*.pdmp | tar -xf - -C /x11-out && \
  apk info -R emacs-gtk3 | grep '^so:' > /x11-out/x11-so-deps

# Final root (pgtk build plus the X11 binary)
FROM ${BASE_IMAGE} AS rootfs
ARG REPO

COPY --from=x11 /x11-out/ /

RUN \
  echo "**** install packages ****" && \
  apk add --no-cache \
    emacs-pgtk \
    $(cat /x11-so-deps) && \
  rm -f /x11-so-deps && \
  echo "**** customize desktop file ****" && \
  sed -i \
    -e "/^Exec=/c Exec=/bin/sh -c \"\$HOME\/.local\/bin\/proot-apps run ghcr.io/${REPO}:emacs %F\"" \
    -e "s/^Name=Emacs$/Name=Emacs PA/" \
    -e '/^TryExec=/d' \
    /usr/share/applications/emacs.desktop && \
  mv \
    /usr/share/applications/emacs.desktop \
    /usr/share/applications/emacs-pa.desktop && \
  echo "**** set bin name ****" && \
  echo "emacs-pa" > /bin-name && \
  echo "**** cleanup ****" && \
  rm -rf \
    /tmp/*

COPY ./entrypoint /entrypoint
COPY ./install /install
COPY ./remove /remove

FROM scratch
COPY --from=rootfs / /
