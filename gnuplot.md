#GNUPLOT example file
```shell
#Save the graph as a png
set terminal png
set output "/tmp/packets-graph.png"

#Set the x axis information
set xdata time
set timefmt '%H:%M:%S'
set xlabel 'Time'

set ylabel 'Packets/s'
set yrange [0:]

plot '/tmp/eth0-rx.data' using 1:2 title 'received' with lines, \
'/tmp/eth0-tx.data' using 1:2 title 'transmitted' with lines
```