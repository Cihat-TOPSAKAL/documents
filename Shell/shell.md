* Json multi line to single line ;

        $ cat data.json | jq -c '.[]' > export_lines.json    