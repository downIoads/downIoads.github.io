---
title: "How to calculate any weekday"
date: 2023-07-04T20:00:00+02:00
description: Will christmas be a Sunday this year? Let's calculate!
draft: false
tags: [tutorial, go, programming]
---

## Introduction

I once read about a simple way to calculate what day of the week any given date is. All you need to do is remember a few numbers and do basic math in your head. So let's get started!

## What you need to remember

Each weekday has an assigned number:
* Sunday = 0
* Monday = 1
* Tuesday = 2
* Wednesday = 3
* Thursday = 4
* Friday = 5
* Saturday = 6

Also remember the following list for the months:
* January = 6 (If leap year: 5)
* February = 2 (If Leap year: 1)
* March = 2
* April = 5
* May = 0
* June = 3
* July = 5
* August = 1
* September = 4
* October = 6
* November = 2
* December = 4

Every year also has a number code, but you only need to know today's day of the week and you can use this knowledge to get this year's code. And if you know this year's code, you can calculate which day this year's 12-31 will be, so you know which weekday next year's 01-01 will be, and so on...

## Ravioli, ravioli, give me the formuoli!
The formula is: (\<day\> + \<month code\> + \<year code\>) mod 7 = \<weekday code\>.
Let's make this clear with an example. This year's code is 0. Which weekday is the 5th of July? 
5 (Day) + 5 (July code) + 0 (year code) mod 7 = 3 \<weekday code\>. 3 maps to Wednesday, so that means the 5th of July is a Wednesday. Again, if you know the month codes by heart, you only need to remember one more number, which is this year's code. This year's code will be 0 for the entire year (year codes are always constant for the entirety of the year). So let's do another example: Which day of the week will the 20th of November be? Stop reading and try it yourself. The solution is 20 + 2 + 0 mod 7 = 1, so the solution is Monday!

## What are the other yearcodes?
This technique is only impressive if you know the codes of many years by heart. So here is a list to get you started:
* 1800 :  3
* 1801 :  4
* 1802 :  5
* 1803 :  6
* 1804 (Leap year):  1
* 1805 :  2
* 1806 :  3
* 1807 :  4
* 1808 (Leap year):  6
* 1809 :  0
* 1810 :  1
* 1811 :  2
* 1812 (Leap year):  4
* 1813 :  5
* 1814 :  6
* 1815 :  0
* 1816 (Leap year):  2
* 1817 :  3
* 1818 :  4
* 1819 :  5
* 1820 (Leap year):  0
* 1821 :  1
* 1822 :  2
* 1823 :  3
* 1824 (Leap year):  5
* 1825 :  6
* 1826 :  0
* 1827 :  1
* 1828 (Leap year):  3
* 1829 :  4
* 1830 :  5
* 1831 :  6
* 1832 (Leap year):  1
* 1833 :  2
* 1834 :  3
* 1835 :  4
* 1836 (Leap year):  6
* 1837 :  0
* 1838 :  1
* 1839 :  2
* 1840 (Leap year):  4
* 1841 :  5
* 1842 :  6
* 1843 :  0
* 1844 (Leap year):  2
* 1845 :  3
* 1846 :  4
* 1847 :  5
* 1848 (Leap year):  0
* 1849 :  1
* 1850 :  2
* 1851 :  3
* 1852 (Leap year):  5
* 1853 :  6
* 1854 :  0
* 1855 :  1
* 1856 (Leap year):  3
* 1857 :  4
* 1858 :  5
* 1859 :  6
* 1860 (Leap year):  1
* 1861 :  2
* 1862 :  3
* 1863 :  4
* 1864 (Leap year):  6
* 1865 :  0
* 1866 :  1
* 1867 :  2
* 1868 (Leap year):  4
* 1869 :  5
* 1870 :  6
* 1871 :  0
* 1872 (Leap year):  2
* 1873 :  3
* 1874 :  4
* 1875 :  5
* 1876 (Leap year):  0
* 1877 :  1
* 1878 :  2
* 1879 :  3
* 1880 (Leap year):  5
* 1881 :  6
* 1882 :  0
* 1883 :  1
* 1884 (Leap year):  3
* 1885 :  4
* 1886 :  5
* 1887 :  6
* 1888 (Leap year):  1
* 1889 :  2
* 1890 :  3
* 1891 :  4
* 1892 (Leap year):  6
* 1893 :  0
* 1894 :  1
* 1895 :  2
* 1896 (Leap year):  4
* 1897 :  5
* 1898 :  6
* 1899 :  0
* 1900 :  1
* 1901 :  2
* 1902 :  3
* 1903 :  4
* 1904 (Leap year):  6
* 1905 :  0
* 1906 :  1
* 1907 :  2
* 1908 (Leap year):  4
* 1909 :  5
* 1910 :  6
* 1911 :  0
* 1912 (Leap year):  2
* 1913 :  3
* 1914 :  4
* 1915 :  5
* 1916 (Leap year):  0
* 1917 :  1
* 1918 :  2
* 1919 :  3
* 1920 (Leap year):  5
* 1921 :  6
* 1922 :  0
* 1923 :  1
* 1924 (Leap year):  3
* 1925 :  4
* 1926 :  5
* 1927 :  6
* 1928 (Leap year):  1
* 1929 :  2
* 1930 :  3
* 1931 :  4
* 1932 (Leap year):  6
* 1933 :  0
* 1934 :  1
* 1935 :  2
* 1936 (Leap year):  4
* 1937 :  5
* 1938 :  6
* 1939 :  0
* 1940 (Leap year):  2
* 1941 :  3
* 1942 :  4
* 1943 :  5
* 1944 (Leap year):  0
* 1945 :  1
* 1946 :  2
* 1947 :  3
* 1948 (Leap year):  5
* 1949 :  6
* 1950 :  0
* 1951 :  1
* 1952 (Leap year):  3
* 1953 :  4
* 1954 :  5
* 1955 :  6
* 1956 (Leap year):  1
* 1957 :  2
* 1958 :  3
* 1959 :  4
* 1960 (Leap year):  6
* 1961 :  0
* 1962 :  1
* 1963 :  2
* 1964 (Leap year):  4
* 1965 :  5
* 1966 :  6
* 1967 :  0
* 1968 (Leap year):  2
* 1969 :  3
* 1970 :  4
* 1971 :  5
* 1972 (Leap year):  0
* 1973 :  1
* 1974 :  2
* 1975 :  3
* 1976 (Leap year):  5
* 1977 :  6
* 1978 :  0
* 1979 :  1
* 1980 (Leap year):  3
* 1981 :  4
* 1982 :  5
* 1983 :  6
* 1984 (Leap year):  1
* 1985 :  2
* 1986 :  3
* 1987 :  4
* 1988 (Leap year):  6
* 1989 :  0
* 1990 :  1
* 1991 :  2
* 1992 (Leap year):  4
* 1993 :  5
* 1994 :  6
* 1995 :  0
* 1996 (Leap year):  2
* 1997 :  3
* 1998 :  4
* 1999 :  5
* 2000 (Leap year) :  0 
* 2001 :  1
* 2002 :  2
* 2003 :  3
* 2004 (Leap year) :  5
* 2005 :  6
* 2006 :  0
* 2007 :  1
* 2008 (Leap year):  3
* 2009 :  4
* 2010 :  5
* 2011 :  6
* 2012 (Leap year):  0
* 2013 :  1
* 2014 :  3
* 2015 :  4
* 2016 (Leap year):  6
* 2017 :  0
* 2018 :  1
* 2019 :  2
* 2020 (Leap year):  4
* 2021 :  5
* 2022 :  6
* 2023 :  0
* 2024 (Leap year):  2
* 2025 :  3
* 2026 :  4
* 2027 :  5
* 2028 (Leap year):  0
* 2029 :  1
* 2030 :  2
* 2031 :  3
* 2032 (Leap year):  5
* 2033 :  6
* 2034 :  0
* 2035 :  1
* 2036 (Leap year):  3
* 2037 :  4
* 2038 :  5
* 2039 :  6

The program used to generate this list can be found here:
## Go

```go

package main

import (
	"fmt"
	"time"
)

func get_weekday(day, month, year uint) uint {
	date := time.Date(int(year), time.Month(month), int(day), 0, 0, 0, 0, time.UTC)
	weekday := uint(date.Weekday())
	return weekday
}

func get_yearcode(weekday uint) uint {
	var yearcode uint;
	
	for i:=0; i < 7; i++ {
		yearcode = uint((1 + 1 + i) % 7)
		if (yearcode == weekday){
			return yearcode
		}
	}
	
	return 7;	// cant happen but we need some return
	
}

func is_leap_year(year uint) bool {
    if year%4 == 0 && (year%100 != 0 || year%400 == 0) {
        return true
    }
    return false
}

func main() {
	// uses the first day of every year as anchor point (might as well take another day)
	const (
		day = uint(1)
		month = uint(1)
	)

	// adjust this depending on the year codes you care about
	for y:= 1800; y<2040; y++ {
		year := uint(y)
		var is_leap_year bool = is_leap_year(year)
		weekday := get_weekday(day, month, year)
		yearcode := get_yearcode(weekday)
		if (is_leap_year) {
			yearcode++
			if (yearcode == 7) {
				yearcode = 0
			}
			fmt.Println("*", year, "(Leap year): ", yearcode)
			continue
		} else {
			fmt.Println("*", year, ": ", yearcode)
		}
		
	}

}


```

## Final words
Congratulations, you now know how to calculate any weekday as long as you remember the codes above and have a basic understand of determining whether a given year is a leap year or not. You also understand the system well enough to be able to calculate next year's code! The code for 2024 will be 2.