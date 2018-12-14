注意

- 某些可配置化的参数，没有100%的把握，不要去随意修改，修改后的结果（副作用）很可能比你想象中的要严重很多
- 其实大多数配置的默认值已经是最佳配置了，所以最好不要去动太多配置，甚至可以忘记某些配置的存在  

   -摘抄于<高性能MySQL>
   
   
   
 ### innodb_buffer_pool_size
    
    作用：这个参数主要作用是缓存innodb表的索引，数据，插入数据时的缓冲
    默认值：128m（show global variables like 'innodb_buffer_pool_size';）
    设置方法：
    
    my.cnf文件
    
    innodb_buffer_pool_size = 6G
   
   
   与MyISAM不同，InnoDB使用缓冲池来缓存索引和# row数据。设置的越大，对表中的数据进行#访问所需的磁盘I/O就越少。在专用数据库服务器上，您可以将这个#参数设置为机器物理内存大小的80%。但是，不要将它设置为#太大，因为物理内存的竞争可能会导致操作系统中的分页。注意，在32位系统上，每个进程的用户级内存可能限制在2-3.5G，所以不要设置太高

### max_length_for_ sort_data
 在 MySQL 中,决定使用老式排序算法还是改进版排序算法是通过参数 max_length_for_ sort_data 来决定的。当所有返回字段的最大长度小于这个参数值时,MySQL 就会选择改进后的排序算法,反之,则选择老式的算法。所以,如果有充足的内存让MySQL 存放须要返回的非排序字段,就可以加大这个参数的值来让 MySQL 选择使用改进版的排序算法
### Sort_Buffer_Size
 这个值如果过小的话,再加上你一次返回的条数过多,那么很可能就会分很多次进行排序,然后最后将每次的排序结果再串联起来,这样就会更慢,增大 sort_buffer_size 并不是为了让 MySQL选择改进版的排序算法,而是为了让MySQL尽量减少在排序过程中对须要排序的数据进行分段,因为分段会造成 MySQL 不得不使用临时表来进行交换排序。
 但是这个值不是越大越好：
 1. Sort_Buffer_Size 是一个connection级参数,在每个connection第一次需要使用这个buffer的时候,一次性分配设置的内存。
 2. Sort_Buffer_Size 并不是越大越好,由于是connection级的参数,过大的设置+高并发可能会耗尽系统内存资源。
 3. 据说Sort_Buffer_Size 超过2M的时候,就会使用mmap() 而不是 malloc() 来进行内存分配,导致效率降低。
 ### key_buffer_size
 
key_buffer_size是对MyISAM表性能影响最大的一个参数。key_buffer_size指定索引缓冲区的大小，它决定索引处理的速度，尤其是索引读的速度
可以检查状态值Key_read_requests和Key_reads，即可知道key_buffer_size设置是否合理。比例key_reads / key_read_requests应该尽可能的低，至少是1:100，1:1000更好
### table_cache
与max_connections 相关。

table_cache指示表高速缓存的大小。当Mysql访问一个表时，如果在Mysql表缓冲区中还有空间，那么这个表就被打开并放入表缓冲区，这样做的好处是可以更快速地访问表中的内容。一般来说，可以通过查看数据库运行峰值时间的状态值Open_tables和Opened_tables，用以判断是否需要增加table_cache的值，
如果Opened_tables远大于Open_tables，并且Open_tables很接近table_cache，那么就说明table_cache偏小