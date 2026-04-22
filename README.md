grep -RniE "GPL-2\.0-only|GPL-3\.0-only|GNU General Public License" . --binary-files=without-match --exclude-dir=.git --exclude-dir=build --exclude-dir=tools --exclude-dir=third_party
