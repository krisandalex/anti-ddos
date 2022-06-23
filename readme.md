##create ipset
ipset create myset-ip hash:ip

## add to ip set from log, variable params is '-n 100' and 'if($1>6'
while true; do tail -n 100 /home/www/logs/ssl_access_log | grep "GET / HTTP" | cut -d " " -f1 | sort | uniq -c | awk '{if($1>6){print $2}}' | xargs -tl -I _ ipset -A blacklist _; sleep 1; done

## add to ip set from netstat, variable params is 'if($1>4)' 
while true; do netstat -ntup | grep SYN | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort | awk '{if($1>4){print $2}}' | xargs -tl -I _ ipset -A blacklist _; sleep 1; done

##iptables rule
-A INPUT -m set --match-set blacklist src -j DROP
