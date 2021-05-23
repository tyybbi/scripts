#!/usr/bin/env bash

set -o errexit
set -o noclobber
set -o nounset

SCRIPTNAME="distrostate"
VERSION="1.2.0"
OS_INFO=$(awk -F'=' '/^NAME/ { print $2 }' /etc/os-release | tr -d '"')
OS_VERSION=$(cat /etc/debian_version)
OS_NAME=$(awk -F'=' '/VERSION_CODENAME/ { print $2 }' /etc/os-release | tr -d '"')

H1_SEP="=========================================================="
H2_SEP="----------------------------------------------------------"

usage() {
    cat << EOF
NAME
    $SCRIPTNAME - check how mixed (or unmixed) your system is

SYNOPSIS
    $SCRIPTNAME [OPTION]

DESCRIPTION
    The script uses apt-show-versions to gather information about
    package repositories (names, quantity and percent of the packages in system)
    used in the current Debian install.

    -m, --show-missing  Print package names that are not found in any
                        repository. These normally consist of obsolete packages
                        or packages installed outside APT.

    -h, --help          Show usage and exit.
    -V, --version       Show version info and exit.

EOF
}

version() {
    echo "$SCRIPTNAME" "$VERSION"
    exit 0
}

show_obsolete_pkgs() {
    LIST_OBSOLETE=$(grep "installed: No available version\|newer than" "$TMPFILE" | awk '{print $1}')
    if [ "$LIST_OBSOLETE" == "" ]; then
        return
    fi

    echo "Packages not in any repo"
    for i in $LIST_OBSOLETE ; do
        printf "\t%s\n" "$i"
    done
    echo "$H2_SEP"
}

cleanup() {
    rm "$TMPFILE"
}

if [ "$#" -ge 1 ] && [ "$1" == "-h" -o "$1" == "--help" ]; then
    usage
    exit 0
fi

if [ "$#" -ge 1 ] && [ "$1" == "-V" -o "$1" == "--version" ]; then
    version
    exit 0
fi

# Check the "dependencies" first
if [ ! -x /usr/bin/apt-show-versions ]; then
    echo "Install apt-show-versions first."
    exit 1
fi

if [ ! -x /usr/bin/bc ]; then
    echo "Install bc first."
    exit 1
fi

TMPFILE=$(mktemp -p "$HOME" "$SCRIPTNAME".XXXXXXXX --suffix=.tmp)
apt-show-versions | grep -F -v " not installed" >| "$TMPFILE"

PKGS_TOTAL=$(wc -l "$TMPFILE" | cut -d" " -f1)
PKGS_OBSOLETE=$(grep "installed: No available version\|newer than" "$TMPFILE" | wc -l)

REPO_NAMES=$(awk -F"/" '{ print $2 }' "$TMPFILE" | cut -d" " -f1 | sort -u | sed '/^$/d')

echo "$H1_SEP"
echo "Package status of $HOSTNAME - ${OS_INFO} $OS_VERSION (${OS_NAME})"
echo "$H2_SEP"

first_run=true

for line in $REPO_NAMES
do
    NUM_PKGS=$(grep -c "/$line" "$TMPFILE")
    PERCENT=$(echo "scale=5; ($NUM_PKGS/$PKGS_TOTAL)*100" | bc -lq | xargs printf "%1.2f")
    if [ $first_run = true ]; then
        printf "%6s %% \t %-10s \t %9d \t packages\n" "$PERCENT" "$line" "$NUM_PKGS"
    else
        printf "%6s %% \t %-10s \t %d \t    \"   \n" "$PERCENT" "$line" "$NUM_PKGS"
    fi
    first_run=false
done

if [ "$PKGS_OBSOLETE" -ne 0 ]; then
    OBS_PERCENT=$(echo "scale=5; ($PKGS_OBSOLETE/$PKGS_TOTAL)*100" | bc -lq | xargs printf "%1.2f")
    printf "%6s %% \t not in any repo \t %d \t    \"   \n" "$OBS_PERCENT" "$PKGS_OBSOLETE"
fi

echo "$H2_SEP"

if [ "$#" -ge 1 ] && [ "$1" == "-m" -o "$1" == "--show-missing" ]; then
    show_obsolete_pkgs
fi

printf "100.00 %% \t total \t %17d \t packages\n" "$PKGS_TOTAL"
echo "$H2_SEP"

trap cleanup EXIT

exit 0

