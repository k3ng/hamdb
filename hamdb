#!/bin/bash
 
# Hamdb
# Copyright (C) 2001-2023 Anthony Good
 
# This program is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License
# as published by the Free Software Foundation; either version 2
# of the License, or (at your option) any later version.
 
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
 
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place - Suite 330, Boston, MA  02111-1307, USA.
 
# You may contact the author at anthony.good@gmail.com

# To create the database user:
#
# sudo mysql 
#
# CREATE USER hamdb@localhost IDENTIFIED BY 'yourpassword'
#
# GRANT ALL PRIVILEGES ON *.* TO 'hamdb'@localhost IDENTIFIED BY 'yourpassword';

STOREDIR=~/hamdb.temp
VERSION=2023.10.18
CONFIGFILE=~/.hamdb.cnf
#DBPOPULATEDTAGFILE=~/.hamdb.populated
DEBUG=0
WGET=/usr/bin/wget
UNZIP=/usr/bin/unzip
DAY=`date -d "-1 day" | tr '[:upper:]' '[:lower:]' | sed 's/\([a-z]*\).*/\1/'`

# if today is Monday, get the full file, otherwise get a smaller daily update file
if [ ${DAY} == sun ]; then
  FULLIMPORT=1
  FCC_FILE_LOCATION=https://data.fcc.gov/download/pub/uls/complete//l_amat.zip
  DOWNLOAD_FILE=l_amat.zip
else
  FULLIMPORT=0
  FCC_FILE_LOCATION=https://data.fcc.gov/download/pub/uls/daily//l_am_${DAY}.zip
  DOWNLOAD_FILE=l_am_${DAY}.zip
fi


if [ ${DEBUG} -eq 1 ]; then
  echo "In debug mode..."
fi
 
if [ -f ${CONFIGFILE} ]; then
  . ${CONFIGFILE}
else
  echo -n "Hamdb configuration file ${CONFIGFILE} not found.  Would you like to create the file? [y/N]"
  read userinput
  echo -n "Enter MySQL username: "
  read MYSQLUSERNAME
  echo -n "Enter MySQL user's password: "
  read MYSQLPASSWD
  #echo "show databases;" | mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
  echo "#!/bin/sh" > ${CONFIGFILE}
  echo "MYSQLUSERNAME=$MYSQLUSERNAME" >> ${CONFIGFILE}
  echo "MYSQLPASSWD=$MYSQLPASSWD" >> ${CONFIGFILE}
  chmod 700 ${CONFIGFILE}
fi
 
if [ ${DEBUG} -eq 1 ]; then
  echo "Username is ${MYSQLUSERNAME}"
  echo "Password is ${MYSQLPASSWD}"
  echo "\s" | mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}|grep "Server version:" | awk '{print $3}'
fi
 
if [ `echo "show databases;"|mysql --user=hamdb --password=${MYSQLPASSWD}|grep -c fcc_amateur` -eq 0 ]; then
  echo "No database found; creating..."
  echo "create database fcc_amateur;"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
  echo "use fcc_amateur;create table en (fccid int not null, callsign varchar(8) not null, primary key(fccid),full_name varchar(32),first varchar(20),middle varchar(1), last varchar(20), address1 varchar(64), city varchar(20), state varchar(2), zip varchar(10), index idx_zip (zip), index idx_callsign (callsign));"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
  echo "use fcc_amateur;create table am (fccid int not null, callsign varchar(8) not null, primary key(fccid), class varchar(1), col4 varchar(1), col5 varchar(2), col6 varchar(3), former_call varchar(8), former_class varchar(1), index idx_callsign (callsign), index idx_class(class));"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
  echo "use fcc_amateur;create table hd (fccid int not null, callsign varchar(8) not null, primary key(fccid), status varchar(1), index idx_callsign (callsign), index idx_status (status) );"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
fi

if [ `echo "use fcc_amateur;select COUNT(fccid) from en;"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD} -s` -eq 0 ]; then
  FULLIMPORT=1
  FCC_FILE_LOCATION=https://data.fcc.gov/download/pub/uls/complete//l_amat.zip
  DOWNLOAD_FILE=l_amat.zip
fi

if [ ${DEBUG} -eq 1 ]; then
  echo "FCC_FILE_LOCATION: $FCC_FILE_LOCATION"
  echo "DOWNLOAD_FILE: $DOWNLOAD_FILE"
  echo "DAY: $DAY"
  echo "FULLIMPORT: $FULLIMPORT"
fi

case "$1" in
 
  getfccfile)
      echo "Downloading database from FCC..."
      $WGET -nd -d --header="Accept: text/html" --user-agent="Mozilla/5.0 (Macintosh; Intel Mac OS X 10.8; rv:21.0) Gecko/20100101 Firefox/21.0" $FCC_FILE_LOCATION
  ;;
 
  populatedb | populatedatabase | updatedatabase | updatedb | update)
    if [ ! -d ${STOREDIR} ]; then
      mkdir ${STOREDIR}
    fi
    cd ${STOREDIR}
 
    if [ ! -f "${DOWNLOAD_FILE}" ]; then
      echo "Downloading database from FCC..."
      $WGET -nd -d --header="Accept: text/html" --user-agent="Mozilla/5.0 (Macintosh; Intel Mac OS X 10.8; rv:21.0) Gecko/20100101 Firefox/21.0" $FCC_FILE_LOCATION
 
    fi
 
    if [ ! -f EN.dat ]; then
      echo "Unzipping database file EN.dat..."
      ${UNZIP} ./${DOWNLOAD_FILE} EN.dat
    fi
 
    if [ ! -f AM.dat ]; then
      echo "Unzipping database file AM.dat..."
      ${UNZIP} ./${DOWNLOAD_FILE} AM.dat
    fi
 
    if [ ! -f HD.dat ]; then
      echo "Unzipping database file HD.dat..."
      ${UNZIP} ./${DOWNLOAD_FILE} HD.dat
    fi
 
     if [ ! -f en_temp.txt ]; then
      echo "Creating temporary file for table en..."
      cat ./EN.dat |sed s/[\\\"]//g|awk -F "|" '{print "\""$2"\",\""$5"\",\""$8"\",\""$9"\",\""$10"\",\""$11"\",\""$16"\",\""$17"\",\""$18"\",\""$19"\""}'>./en_temp.txt
    fi
 
    if [ ! -f am_temp.txt ]; then
      echo "Creating temporary file for table am..."
      cat ./AM.dat |sed s/[\\\"]//g|awk -F "|" '{print "\""$2"\",\""$5"\",\""$6"\",\""$7"\",\""$8"\",\""$10"\",\""$16"\",\""$17"\""}'>./am_temp.txt
    fi
 
    if [ ! -f hd_temp.txt ]; then
      echo "Creating temporary file for table hd..."
      cat ./HD.dat |sed s/[\\\"]//g|awk -F "|" '{print "\""$2"\",\""$5"\",\""$6"\""}'>./hd_temp.txt
    fi
 
    if [ ${FULLIMPORT} -eq 1 ]; then

      echo "Doing full database import..."
 
      echo "Creating temporary tables in Mysql..."
 
      echo "use fcc_amateur;drop table if exists en_temp;create table en_temp (fccid int not null, callsign varchar(8) not null, primary key(fccid),full_name varchar(64),first varchar(20),middle varchar(1), last varchar(20), address1 varchar(64), city varchar(20), state varchar(2), zip varchar(10), index idx_zip (zip), index idx_callsign (callsign));"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
 
      #echo "use fcc_amateur;drop table if exists en_temp;create table en_temp (fccid int, callsign varchar(8) not null, primary key(callsign),full_name varchar(64),first varchar(20),middle varchar(1), last varchar(20), address1 varchar(64), city varchar(20), state varchar(2), zip varchar(10), index idx_zip (zip), index idx_fccid (fccid));"|mysql --user=${MYSQLUSERNAME} --password=$MYSQLPASSWD
 
      echo "use fcc_amateur;drop table if exists am_temp;create table am_temp (fccid int not null, callsign varchar(8) not null, primary key(fccid), class varchar(1), col4 varchar(1), col5 varchar(2), col6 varchar(3), former_call varchar(8), former_class varchar(1), index idx_callsign (callsign), index idx_class(class));"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
 
      #echo "use fcc_amateur;drop table if exists am_temp;create table am_temp (fccid int, callsign varchar(8) not null, primary key(callsign), class varchar(1), col4 varchar(1), col5 varchar(2), col6 varchar(3), former_call varchar(8), former_class varchar(1), index idx_fccid (fccid), index idx_class(class));"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
 
      echo "use fcc_amateur;drop table if exists hd_temp;create table hd_temp (fccid int not null, callsign varchar(8) not null, primary key(fccid), status varchar(1), index idx_callsign (callsign), index idx_status (status) );"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
 
      echo "Populating database table en..."

      mysqlimport --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD} --fields-terminated-by=',' --fields-enclosed-by='\"' --lines-terminated-by='\n' --local fcc_amateur ${STOREDIR}/en_temp.txt

      echo "use fcc_amateur;rename table en to en_old, en_temp to en;drop table en_old;"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
 
      echo "Populating database table am..."

      mysqlimport --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD} --fields-terminated-by=',' --fields-enclosed-by='\"' --lines-terminated-by='\n' --local fcc_amateur ${STOREDIR}/am_temp.txt
 
      echo "use fcc_amateur;rename table am to am_old, am_temp to am;drop table am_old;"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
 
      echo "Populating database table hd..."
 
      mysqlimport --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD} --fields-terminated-by=',' --fields-enclosed-by='\"' --lines-terminated-by='\n' --local fcc_amateur ${STOREDIR}/hd_temp.txt

      echo "use fcc_amateur;rename table hd to hd_old, hd_temp to hd;drop table hd_old;"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}

    else

      echo "Doing update with a daily update file..."

      # mysqlimport uses the filename to determine the table name (dumb) 
      mv ${STOREDIR}/en_temp.txt ${STOREDIR}/en.txt
      mv ${STOREDIR}/am_temp.txt ${STOREDIR}/am.txt
      mv ${STOREDIR}/hd_temp.txt ${STOREDIR}/hd.txt
 
      echo "Updating database table en..."
    
      mysqlimport --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD} --fields-terminated-by=',' --fields-enclosed-by='\"' --lines-terminated-by='\n' --local --replace fcc_amateur ${STOREDIR}/en.txt
 
      echo "Updating database table am..."
 
      mysqlimport --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD} --fields-terminated-by=',' --fields-enclosed-by='\"' --lines-terminated-by='\n' --local --replace fcc_amateur ${STOREDIR}/am.txt
 
      echo "Updating database table hd..."
 
      mysqlimport --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD} --fields-terminated-by=',' --fields-enclosed-by='\"' --lines-terminated-by='\n' --local --replace fcc_amateur ${STOREDIR}/hd.txt
 
    fi
    if [ ${DEBUG} -eq 0 ]; then
      echo "Cleaning up..."
      rm -f ${STOREDIR}/*_temp.txt
      rm -f ${STOREDIR}/en.txt
      rm -f ${STOREDIR}/am.txt
      rm -f ${STOREDIR}/hd.txt
      rm -f ${STOREDIR}/*.dat
      rm -f ${STOREDIR}/*.zip
      rm -f ${STOREDIR}/counts
      cd -
      rmdir ${STOREDIR}
    fi
 
    echo "Done..."
 
    ;;


  cleanup) 
    echo "Cleaning up..."
    cd ${STOREDIR} 
    rm -f ${STOREDIR}/*_temp.txt
    rm -f ${STOREDIR}/en.txt
    rm -f ${STOREDIR}/am.txt
    rm -f ${STOREDIR}/hd.txt
    rm -f ${STOREDIR}/*.dat
    rm -f ${STOREDIR}/*.zip
    rm -f ${STOREDIR}/counts
    cd -
    rmdir ${STOREDIR}
  ;; 
  makedatabase | makedb)
 
    echo "Creating database fcc_amateur...";
 
    echo "create database fcc_amateur;"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
 
    echo "use fcc_amateur;create table en (fccid int not null, callsign varchar(8) not null, primary key(fccid),full_name varchar(32),first varchar(20),middle varchar(1), last varchar(20), address1 varchar(64), city varchar(20), state varchar(2), zip varchar(10), index idx_zip (zip), index idx_callsign (callsign));"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
 
    echo "use fcc_amateur;create table am (fccid int not null, callsign varchar(8) not null, primary key(fccid), class varchar(1), col4 varchar(1), col5 varchar(2), col6 varchar(3), former_call varchar(8), former_class varchar(1), index idx_callsign (callsign), index idx_class(class));"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
 
    echo "use fcc_amateur;create table hd (fccid int not null, callsign varchar(8) not null, primary key(fccid), status varchar(1), index idx_callsign (callsign), index idx_status (status) );"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
 
  ;;
 
  removedatabase | removedb)
    echo "drop database fcc_amateur;"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
    rm -f ${DBPOPULATEDTAGFILE}
  ;;
 
  removetables)
    echo "use fcc_amateur;drop table if exists en;drop table if exists am;drop table if exists hd;drop table if exists en_temp;drop table if exists am_temp;drop table if exists hd_temp"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
  ;;
 
  zipcodebatch)
    echo "use fcc_amateur;select en.callsign,class,first,last,address1,city,state,zip from en, am, hd  where en.fccid=am.fccid and en.fccid=hd.fccid and hd.status=\"A\" and zip like \"$2%\";" | mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD} -N --batch | awk -F "\t" '{print $1","$2","$3","$4","$5","$6","$7","$8}'
  ;;
 
  dumptableen)
    echo "use fcc_amateur;select * from en;"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
  ;;
 
  dbcount | count | counts)
    echo "Table en count: "
    echo "use fcc_amateur;select COUNT(fccid) from en;"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
    echo "Table am count: "
    echo "use fcc_amateur;select COUNT(fccid) from am;"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
    echo "Table hd count: "
    echo "use fcc_amateur;select COUNT(fccid) from hd;"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
    echo "Table hd status counts: "
    echo "use fcc_amateur;select status, count(fccid) from hd group by status;"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
  ;;
 
  -h | --help | help)
    echo "Help:"
    ;;
 
  -v | --version | version)
    echo ${VERSION}
    ;;
 
  like)
    CALLSIGN=`echo $2|sed y/abcdefghijklmnopqrstuvwxyz/ABCDEFGHIJKLMNOPQRSTUVWXYZ/`
    echo "use fcc_amateur;select en.callsign, am.class, full_name, address1, city, state, zip, former_call from en, am, hd where en.fccid=am.fccid and en.fccid=hd.fccid and hd.status=\"A\" and en.callsign like  \"$CALLSIGN%\";"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
  ;;
 
  *)
    CALLSIGN=`echo $1|sed y/abcdefghijklmnopqrstuvwxyz/ABCDEFGHIJKLMNOPQRSTUVWXYZ/`
    echo "use fcc_amateur;select en.callsign, am.class, full_name, address1, city, state, zip, former_call from en, am, hd where en.fccid=am.fccid and en.fccid=hd.fccid and hd.status=\"A\" and en.callsign=\"$CALLSIGN\";"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
    #echo "use fcc_amateur;select * from en where en.callsign=\"$CALLSIGN\";"|mysql --user=${MYSQLUSERNAME} --password=${MYSQLPASSWD}
    exit 0
esac
