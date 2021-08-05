PARALLEL_MAKE = ""

all: main.o 
	${CC} ${LDFLAGS} main.o -o fpga2wifi_bin
main.o: src/main.c
	${CC} -I . -c src/main.c
clean:
	rm -rf *.o
	rm fpga2wifi_bin