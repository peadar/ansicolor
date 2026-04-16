#!/usr/bin/env python3
import sys, os, math
from contextlib import contextmanager
import argparse
import collections
import colorsys

linesOutput = 0
def o(s):
    global linesOutput
    sys.stdout.write(s)
    linesOutput += s.count("\n")

ESC = "\033"
CSI = f"{ESC}["
STOP = f"{CSI}0m"
CLS =  f"{CSI}2J"
HOME =  f"{CSI}H"

colincr = 0

Args = collections.namedtuple( "Args", [ "rows", "cols", "noalt" ] )
args = Args( cols=80, rows=24, noalt=False )

@contextmanager
def EightBitBackground(color):
    o(f"{CSI}48;5;{color}m")
    yield
    o(STOP)

@contextmanager
def EightBitForeground(color):
    o(f"{CSI}38;5;{color}m")
    yield
    o(STOP)

@contextmanager
def AltScreen():
    if not args.noalt:
        o(f"{CSI}?1049h")
    yield
    if not args.noalt:
        o(f"{CSI}?1049l")

@contextmanager
def TrueColor(r, g, b):
    o(f"{CSI}48;2;{r};{g};{b}m")
    yield
    o(STOP)

def cells(index, count, space):
    # width of the Nth field of count. In theory, (index/count) * cols, but we need to distribute the fractional bits 
    per = float(space) / count
    start = index * per
    end = (index + 1) * per
    return int ( math.floor(end) - math.floor(start) ) 

def widthof(index, count):
    return cells(index, count, args.cols)

def heightof(index, count):
    return cells(index, count, args.rows)

def basic():
    o("Basic ANSI 16-color\n")
    for ctx in ( EightBitForeground, EightBitBackground ):
        # Show first 16 separately
        for color in range(16):
            with ctx(color):
                o(f"{color :>{widthof(color,16)-1}} ")
        o("\n")
    o("\nRemaining 8-bit color pallette\n")
    for ctx in ( EightBitForeground, EightBitBackground ):
        for colormajor in range(16, 256, 24):
            for colorminor in range(24):
                with ctx(colormajor+colorminor):
                    o(f"{colorminor+colormajor :>{widthof(colorminor,24)}}")
            o("\n")

def primaries():
    o("\nTrue color - primaries\n")
    for m in [ (1,0,0), (0,1,0), (0,0,1), (1,1,0), (1,0,1), (0,1,1), (1,1,1) ]:
        intensity = 0
        if linesOutput >= args.rows - 1:
            break
        while intensity <= 255:
            i = int(intensity)
            with TrueColor(*((a * b) for a, b in zip(m, (i, i, i)))):
                o(" ")
            intensity += colincr
        o("\n")

def rainbow():
    rainbowlines = min(args.rows - linesOutput - 2, 256)
    o("Rainbow\n")
    for row in range(rainbowlines):
        for col in range(args.cols):
            h = col / (args.cols)
            v = row / rainbowlines
            rgb = colorsys.hsv_to_rgb(h, 1.0, v)
            with TrueColor( *(int(component * 255) for component in rgb )):
               o(" ")
        o("\n")

def testChart():
    basic()
    primaries()
    rainbow()

def main():
    global args, colincr
    argp = argparse.ArgumentParser()
    try:
        sz = os.get_terminal_size()
        cols, rows = sz.columns, sz.lines
    except:
        cols, rows = 80, 24
    argp.add_argument("--cols", type=int, default=cols)
    argp.add_argument("--rows", type=int, default=rows)
    argp.add_argument("--noalt", action='store_true', default=not sys.stdout.isatty())
    args = argp.parse_args()

    colincr = 256 / args.cols
    with AltScreen():
        if not args.noalt:
            o(CLS + HOME)
        testChart()
        if not args.noalt:
            sys.stdin.readline()

if __name__ == "__main__":
    main()
