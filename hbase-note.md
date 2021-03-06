# How the Components Work Together
	

## HBase First Read or Write
	HBase Catalog table, META table: location of the regions in the cluster
	ZooKeeper stores the location of the META table.
	
	1. The client gets the Region server that hosts the META table from ZooKeeper.
	2. The client will query the .META. server to get the region server corresponding to the row key it wants to access. The client caches this information along with the META table location.
	3. It will get the Row from the corresponding Region Server.
	
## HBase Meta Table
	This META table is an HBase table that keeps a list of all regions in the system.
	The .META. table is like a b tree.
	The .META. table structure is as follows:
		- Key: region start key,region id
		- Values: RegionServer

## Region Server Components
	A Region Server runs on an HDFS data node and has the following components:
		WAL: Write Ahead Log is a file on the distributed file system. The WAL is used to store new data that hasn't yet been persisted to permanent storage; it is used for recovery in the case of failure.
		BlockCache: is the read cache. It stores frequently read data in memory. Least Recently Used data is evicted when full.
		MemStore: is the write cache. It stores new data which has not yet been written to disk. It is sorted before writing to disk. There is one MemStore per column family per region.
		Hfiles store the rows as sorted KeyValues on disk.
		
		
## HBase Write Steps (1)
	When the client issues a Put request, the first step is to write the data to the write-ahead log, the WAL:
	- Edits are appended to the end of the WAL file that is stored on disk.
	- The WAL is used to recover not-yet-persisted data in case a server crashes.

## HBase Write Steps (2)
	Once the data is written to the WAL, it is placed in the MemStore. Then, the put request acknowledgement returns to the client.

## HBase MemStore
	The MemStore stores updates in memory as sorted KeyValues, the same as it would be stored in an HFile. There is one MemStore per column family. The updates are sorted per column family.

## HBase Region Flush
	When the MemStore accumulates enough data, the entire sorted set is written to a new HFile in HDFS
	HBase uses multiple HFiles per column family, which contain the actual cells, or KeyValue instances.
	There is a limit to the number of column families in HBase
	The highest sequence number is stored as a meta field in each HFile, to reflect where persisting has ended and where to continue.
	sequence number???
	
	 
## HBase HFile
	contains sorted key/values
	written to a new HFile in HDFS
	sequential write
	

## HBase HFile Structure
	contains a multi-layered index like a b+tree
	allows HBase to seek to the data without having to read the whole file
		Key value pairs are stored in increasing order
		Indexes point by row key to the key value data in 64KB “blocks”
		Each block has its own leaf-index
		The last key of each block is put in the intermediate index
		The root index points to the intermediate index
		trailer points points to the meta blocks, written at the end of persisting the data to the file
		The trailer also has information like bloom filters and time range info
		Bloom filters help to skip files that do not contain a certain row key
		The time range info is useful for skipping the file if it is not in the time range the read is looking for
		

## HFile Index
	is loaded when the HFile is opened and kept in memory => what is located in BlockCache
	a single disk seek

## HBase Read Merge
	KeyValue cells corresponding to one row can be
		row cells already persisted are in Hfiles
		recently updated cells are in the MemStore
		recently read cells are in the Block cache
	How does the system get the corresponding cells to return?
		1. looks for the Row cells in the Block cache - the read cache
		2. the scanner looks in the MemStore
		3. If the scanner does not find all of the row cells in the MemStore and Block Cache, then HBase will use the Block Cache indexes and bloom filters to load HFiles into memory, which may contain the target row cells.
	
		
## HBase Read Merge
	read amplification

## HBase Minor Compaction
	automatically pick some smaller HFiles and rewrite them into fewer bigger Hfiles
	performing a merge sort
	improves read performance
	
## HBase Major Compaction
	Major compaction merges and rewrites all the HFiles in a region to one HFile per column family
	drops deleted or expired cells
	since major compaction rewrites all of the files, lots of disk I/O and network traffic might occur during the process
	write amplification
	Major compactions can be scheduled to run automatically

## Region = Contiguous Keys
	A table can be divided horizontally into one or more regions. A region contains a contiguous, sorted range of rows between a start key and an end key
	Each region is 1GB in size (default)
	A region of a table is served to the client by a RegionServer
	A region server can serve about 1,000 regions (which may belong to the same table or different tables)

## Region Split
	Initially there is one region per table. When a region grows too large, it splits into two child regions.
	Both child regions, representing one-half of the original region, are opened in parallel on the same Region server, and then the split is reported to the HMaster
	For load balancing reasons, the HMaster may schedule for new regions to be moved off to other servers.

## Read Load Balancing
	the HMaster may schedule for new regions to be moved off to other servers
	HBase data is local when it is written, but when a region is moved (for load balancing or recovery), it is not local until major compaction.

## HDFS Data Replication
	All writes and Reads are to/from the primary node.
	HDFS replicates the WAL and HFile blocks
	HFile block replication happens automatically
	HBase relies on HDFS to provide the data safety as it stores its files
	When data is written in HDFS, one copy is written locally, and then it is replicated to a secondary node, and a third copy is written to a tertiary node.


## HDFS Data Replication (2)
	

## HBase Crash Recovery
	Zookeeper will determine Node failure when it loses region server heart beats
	The HMaster will then be notified that the Region Server has failed.
	
	The HMaster reassigns the regions from the crashed server to active Region servers
	The HMaster splits the WAL belonging to the crashed region server into separate files and stores these file in the new region servers’ data nodes. 
	Each Region Server then replays the WAL from the respective split WAL, to rebuild the memstore for that region.
	
## Data Recovery
	WAL files contain a list of edits, with one edit representing a single put or delete.
	Edits are written chronologically
	Replaying a WAL is done by reading the WAL, adding and sorting the contained edits to the current MemStore.
	

## Apache HBase Architecture Benefits
	Strong consistency model
	Scales automatically
	Built-in recovery
	Integrated with Hadoop
	

## Apache HBase Has Problems Too…
	Business continuity reliability:
		- WAL replay slow
		- Slow complex crash recovery
		- Major Compaction I/O storms


## BloomFilter:	
	An HBase Bloom Filter is an efficient mechanism to test whether a StoreFile contains a specific row or row-col cell
