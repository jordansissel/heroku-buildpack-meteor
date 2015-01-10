#!/usr/bin/env bash
# USAGE: bin/compile_meteor <build-dir> <cache-dir> <env-dir>

####### Configure environment

set -e  # fail fast

###### NEEDED VARIABLES/DIRECTORIES

BUILD_DIR=$1
CACHE_DIR=$2
ENV_DIR=$3
APP_SOURCE=$BUILD_DIR/app_src
METEOR_HOME=$BUILD_DIR/.meteor_tool
PATH=$METEOR_HOME:$BUILD_DIR/.heroku/node/bin:$PATH
bp_dir=$(cd $(dirname $0); cd ..; pwd)

# Load some convenience functions like status(), echo(), and indent()
source $bp_dir/bin/common.sh

fetch_meteor() {
  # We determine the desired Meteor version from .meteor/release
  METEOR_VERSION=`cat "$APP_SOURCE/.meteor/release" | sed 's/METEOR@//'`

  # if there are old cached versions, we should delete them
  (
    cd $CACHE_DIR
    old_files=`find . -maxdepth 1 -name 'meteor-*.tar.gz' -not -name "meteor-$METEOR_VERSION.tar.gz"`
    for f in $old_files; do rm $f; done
  )

  head "Fetching Meteor $METEOR_VERSION"
  PACKAGE="$CACHE_DIR/meteor-$METEOR_VERSION.tar.gz"
  TARBALL_URL="https://d3sqy0vbqsdhku.cloudfront.net/packages-bootstrap/${METEOR_VERSION}/meteor-bootstrap-os.linux.x86_64.tar.gz"
  if [ ! -e "$PACKAGE" ]; then
    curl -o "$PACKAGE" "$TARBALL_URL" | indent
  else
    info "Using cached Meteor $METEOR_VERSION"
  fi

  head "Unpacking Meteor $METEOR_VERSION"
  tar -xzf "$PACKAGE" -C "$BUILD_DIR"
  mv "$BUILD_DIR/.meteor" "$METEOR_HOME"
  # bomb out if it didn't work, eg no net
  [ ! -x "${METEOR_HOME}/meteor" ] && error "Install failed: No meteor bin found."
  info "Meteor $METEOR_VERSION is installed"
}

build() {
  (
    head "Building Meteor App Bundle"
    cd $APP_SOURCE
    # can't build with these platforms enabled, it looks like
    set +e  # these next few are allowed to exit nonzero
    grep -Fxq ios .meteor/platforms
    if [ $? -eq 0 ]; then
      meteor remove-platform ios 2>&1 | indent
    fi
    grep -Fxq android .meteor/platforms
    if [ $? -eq 0 ]; then
      meteor remove-platform android 2>&1 | indent
    fi
    set -e  # back to failing if any command fails
    meteor build ../build --directory 2>&1 | indent
    head "Installing App's NPM Dependencies"
    (cd ../build/bundle/programs/server && npm install 2>&1 | indent)
    cd $BUILD_DIR && rm -rf $APP_SOURCE
  )
}

[ ! -d $BUILD_DIR ] && mkdir $BUILD_DIR
[ ! -d $CACHE_DIR ] && mkdir $CACHE_DIR

fetch_meteor
build

if [ -f $BUILD_DIR/bin/post_compile ] ; then
    head "Running post_compile hook"
    chmod +x $BUILD_DIR/bin/post_compile
    $BUILD_DIR/bin/post_compile
fi
