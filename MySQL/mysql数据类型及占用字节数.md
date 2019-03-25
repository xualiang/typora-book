数字类型

根据字节数即可算出表示的范围了 

TINYINT                                    1 字节 

SMALLINT                                2 个字节 

MEDIUMINT                              3 个字节 

INT                                           4 个字节 

INTEGER                                   4 个字节 

BIGINT                                      8 个字节 

FLOAT(X)                                  4 如果 X < = 24 或 8 如果 25 < = X < = 53 

FLOAT                                       4 个字节 

DOUBLE                                    8 个字节 

DOUBLE PRECISION                  8 个字节 

REAL                                         8 个字节 

DECIMAL(M,D)                          M字节(D+2 , 如果M < D) 

NUMERIC(M,D)                          M字节(D+2 , 如果M < D)

 

日期和时间类型

DATE                                        3 个字节 

DATETIME                                 8 个字节 

TIMESTAMP                               4 个字节 

TIME                                         3 个字节 

YEAR                                         1 字节

 

字符串类型

CHAR(M)                                        M字节，1 <= M <= 255 

VARCHAR(M)                                 L+1 字节, 在此L <= M和1 <= M <= 255 

TINYBLOB, TINYTEXT                     L+1 字节, 在此L< 2 ^ 8 

BLOB, TEXT                                   L+2 字节, 在此L< 2 ^ 16 

MEDIUMBLOB, MEDIUMTEXT         L+3 字节, 在此L< 2 ^ 24 

LONGBLOB, LONGTEXT                 L+4 字节, 在此L< 2 ^ 32 

ENUM('value1','value2',...)                1 或 2 个字节, 取决于枚举值的数目(最大值65535） 

SET('value1','value2',...)                    1，2，3，4或8个字节, 取决于集合成员的数量(最多64个成员）

