#!/bin/bash
# scripts.sh
# vim: ft=sh

#   __|    \    _ \  |      _ \   __| __ __| __ __|
#  (      _ \     /  |     (   | (_ |    |      |
# \___| _/  _\ _|_\ ____| \___/ \___|   _|     _|

# --------------------------------------
# Description: carlogtt personal scripts
# --------------------------------------


# -------------------------------------
# pstree
# Recursively find all process children
# -------------------------------------
pstree() {
    _find_children() {
        for child in $(sudo ps -e -o "pid,ppid" | awk -v ppid="${1}" '$2 == ppid {print $1}'); do
            echo -e '\033[47m'
            sudo ps -p "${child}" -o "pid,ppid,stat,uid,user,pri,ni,tty,%cpu,%mem,rss,vsz,start,time,command"
            echo -e '\033[0m'

            echo -e -n '\033[36m'
            if [[ $(uname -s) == "Darwin" ]]; then
                echo -e "PID Environment Variables\n $(sudo ps eww -o command ${child} | tr ' ' '\n' | awk '/=/{print}' | tr '\n' ' ')"
            else
                echo -e "PID Environment Variables\n $(sudo /bin/cat /proc/${child}/environ | tr '\0' '\n' | grep -E -v '^0$' | tr '\n' ' ')"
            fi

            echo -e -n '\033[32m'
            sudo lsof -p "${child}" | grep -v -E 'IPv4|IPv6'

            echo -e -n '\033[35m'
            sudo lsof -i -a -p "${child}"
            echo -e -n '\033[0m'

            _find_children "${child}"
        done
        echo -e ""
    }

    _find_parents() {
        local child_pid="${1}"
        local parent_pid=$(sudo ps -p "${child_pid}" -o ppid= | tr -d ' ')
        local parents=()

        # Collect all parent PIDs in a list
        while [ "${parent_pid}" -ne 1 ]; do
            parents+=("${parent_pid}")
            child_pid="${parent_pid}"
            parent_pid=$(sudo ps -p "${child_pid}" -o ppid= | tr -d ' ')
        done
        parents+=("1")

        # Print all parent PIDs in reverse order
        for ((idx=${#parents[@]}-1; idx>=0; idx--)); do
            pid="${parents[@]:${idx}:1}"

            echo -e '\033[47m'
            sudo ps -p "${pid}" -o "pid,ppid,stat,uid,user,pri,ni,tty,%cpu,%mem,rss,vsz,start,time,command"
            echo -e '\033[0m'

            echo -e -n '\033[36m'
            if [[ $(uname -s) == "Darwin" ]]; then
                echo -e "PID Environment Variables\n $(sudo ps eww -o command ${pid} | tr ' ' '\n' | awk '/=/{print}' | tr '\n' ' ')"
            else
                echo -e "PID Environment Variables\n $(sudo /bin/cat /proc/${pid}/environ | tr '\0' '\n' | grep -E -v '^0$' | tr '\n' ' ')"
            fi

            echo -e -n '\033[32m'
            sudo lsof -p "${pid}" | grep -v -E 'IPv4|IPv6'

            echo -e -n '\033[35m'
            sudo lsof -i -a -p "${pid}"
            echo -e -n '\033[0m'
        done
        echo -e ""
    }

    # ENTRY POINT
    if [[ -z "${1}" ]]; then
        echo -e "Usage: pstree <PID>"
        return 1
    fi

    if ! sudo ps -p "${1}" >/dev/null 2>&1; then
        echo -e "Invalid <PID>"
        return 1
    fi

    # Print parent processes
    _find_parents "${1}" | awk 'NF || !blank++'

    # Print target process
    echo -e '\033[43m'
    sudo ps -p "${1}" -o "pid,ppid,stat,uid,user,pri,ni,tty,%cpu,%mem,rss,vsz,start,time,command"
    echo -e '\033[0m'

    echo -e -n '\033[36m'
    if [[ $(uname -s) == "Darwin" ]]; then
        echo -e "PID Environment Variables\n $(sudo ps eww -o command ${1} | tr ' ' '\n' | awk '/=/{print}' | tr '\n' ' ')"
    else
        echo -e "PID Environment Variables\n $(sudo /bin/cat /proc/${1}/environ | tr '\0' '\n' | grep -E -v '^0$' | tr '\n' ' ')"
    fi

    echo -e -n '\033[32m'
    sudo lsof -p "${1}" | grep -v -E 'IPv4|IPv6'

    echo -e -n '\033[35m'
    sudo lsof -i -a -p "${1}"
    echo  -e -n '\033[0m'

    echo -e ""

    # Print child processes
    _find_children "${1}" | awk 'NF || !blank++'
}


# --------------------------------------
# mkextract
# Extract files in their own directories
# --------------------------------------
mkextract() {
    _extract() {
        case $1 in
            *.tar)                        tar -xf          "${1}"              ;;

            *.tar.gz | *.tgz)             tar --gzip -xf   "${1}"              ;;
            *.tar.bz2 | *.tbz2 | *.tbz)   tar --bzip2 -xf  "${1}"              ;;
            *.tar.xz | *.txz)             tar --xz -xf     "${1}"              ;;
            *.tar.lz)                     tar --lzip -xf   "${1}"              ;;
            *.tar.lzma | *.tlz)           tar --lzma -xf   "${1}"              ;;
            *.tar.lzo)                    tar --lzop -xf   "${1}"              ;;
            *.tar.zst | *.tzst)           tar --zstd -xf   "${1}"              ;;
            *.tar.Z)                      tar -Z -xf       "${1}"              ;;

            *.gz)                         gzip -dk         "${1}"              ;;
            *.bz2)                        bzip2 -dk        "${1}"              ;;
            *.xz | *.lzma)                xz -dk           "${1}"              ;;
            *.zst)                        zstd -dk         "${1}"              ;;
            *.Z)                          uncompress       "${1}"              ;;

            *.zip)                        unzip            "${1}"              ;;
            *.7z)                         7z x             "${1}"              ;;
            *.rar)                        7z x             "${1}"              ;;
            *.iso)                        7z x             "${1}"              ;;
            *)                            echo "'${1}' cannot be extracted" ;;
        esac
    }

    # ENTRY POINT
    if [[ -z "${1}" ]]; then
        echo -e "Usage: mkextract <FILE(s)>"
        return 1
    fi

    for file in "$@"
    do
        local basename=$(basename "${file}")
        local ext_dir_name="${basename%%\.*}"
        local dirpath=$(realpath -- "$(dirname "${file}")")
        local filepath=$(realpath -- "${file}")

        if [ -f "${filepath}" ]; then
            mkdir -p -- "${dirpath}/${ext_dir_name}/extracted"
            mv -- "${filepath}" "${dirpath}/${ext_dir_name}"
            cp -- "${dirpath}/${ext_dir_name}/${file}" "${dirpath}/${ext_dir_name}/extract/${file}"
            _extract "${dirpath}/${ext_dir_name}/extract/${file}"
            rm -rf -- "${dirpath}/${ext_dir_name}/extract/${file}"
        else
            echo "'$1' is not a valid file"
        fi
    done
}

