tags: #grep

**Filter for Lines Containing Multiple Patterns**: To find lines that match any of several patterns:

```
cat subs | grep -E "google.com|yahoo.com|bing.com"

```

**Show Only Lines Matching a Pattern (Case-Insensitive)**: To perform a case-insensitive search:

```
cat subs | grep -i "feedproxy"

```

**Show Line Numbers Alongside Matches**: To show the line numbers of matching lines:

```
cat subs | grep -n "feedproxy.ghs.google.com"

```

**Count the Number of Matches**: To count how many lines match the pattern:

```
cat subs | grep -c "feedproxy.ghs.google.com"
```

**Highlight Matches**: To highlight matches in color for better visibility:

```
cat subs | grep --color=always "feedproxy"
```

**Exclude Specific Patterns**: To exclude lines that contain a certain pattern:

```
cat subs | grep -v "feedproxy.ghs.google.com"
```

**Filter Lines Starting or Ending with a Specific Pattern**: To filter lines starting with a specific string:

```
cat subs | grep "^www"

```

To filter lines ending with a specific string:

```
cat subs | grep "google.com$"

```

**Invert Match with Multiple Exclusions**: To exclude lines containing multiple patterns:

```
cat subs | grep -v -e "feedproxy.ghs.google.com" -e "ads.google.com"

```