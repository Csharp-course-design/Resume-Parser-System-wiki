> ### 即，为DAL编写提供一些 `SQL` 编写示例，用以加快项目编写

## 插入代码示例

其中，插入功能分为 *构建 `SQL` 语句*，*实现接口函数*

```csharp

        /// <summary>
        /// 插入指定内容。
        /// </summary>
        /// <param name="item">插入的指定表的数据对象。</param>
        public void Insert(SqlTransaction sqlTransaction, object item)
        {
            InsertReturnID(sqlTransaction, item);
        }

        /// <summary>
        /// 插入指定内容，并返回插入数据的 ID。
        /// </summary>
        /// <param name="item">插入的指定表的数据对象。</param>
        /// <returns>数据的 ID。</returns>
        public string InsertReturnID(SqlTransaction sqlTransaction, object item)
        {
            SqlConnection conn = GetSqlConnection();
            {
                OpenSqlConnection();
                SqlTransaction tx;

                if (sqlTransaction == null)
                    tx = conn.BeginTransaction();
                else
                    tx = sqlTransaction;

                PeopleData Data = (PeopleData)item;

                if (!Data.Base.IsEmpty())
                {
                    try
                    {
                        List<string> fields = new List<string>
                        {
                            "PEB_Name",
                            "PEB_Sex",
                            "PEB_Birthday",
                            "PEB_Job",
                            "PEB_Title",
                            "PEB_Employer",
                        };

                        List<string> datas = new List<string>
                        {
                            Data.Base.PEB_Name,
                            Data.Base.PEB_Sex,
                            Data.Base.PEB_Birthday,
                            Data.Base.PEB_Job,
                            Data.Base.PEB_Title,
                            Data.Base.PEB_Employer

                        };

                        string SQL = BuildInsertSQL("PeopleBase", fields, datas);

                        string newID;

                        using (SqlCommand cmd = new SqlCommand(SQL, conn, tx))
                        {
                            for (int i = 0; i < datas.Count; i++)
                            {
                                cmd.Parameters.AddWithValue($"@param{i}", datas[i]);
                            }


                            newID = cmd.ExecuteScalar()?.ToString();
                        }

                        if (!string.IsNullOrEmpty(newID) && !Data.PrincipalExpand.IsEmpty())
                        {
                            fields = new List<string>
                            {
                                "PEB_ID",
                                "PE_Major",
                                "PE_Speciality",
                                "PE_Engage",
                                "PE_Address",
                                "PE_IsYouth",
                                "PE_OfficePhone",
                                "PE_MobilePhone",
                                "PE_Email",
                            };

                            datas = new List<string>
                            {
                                newID,
                                Data.PrincipalExpand.PE_Major,
                                Data.PrincipalExpand.PE_Speciality,
                                Data.PrincipalExpand.PE_Engage,
                                Data.PrincipalExpand.PE_Address,
                                Data.PrincipalExpand.PE_IsYouth,
                                Data.PrincipalExpand.PE_OfficePhone,
                                Data.PrincipalExpand.PE_MobilePhone,
                                Data.PrincipalExpand.PE_Email,
                            };

                            string expandQuery = BuildInsertSQL("PeoplePrincipalExpandData", fields, datas);

                            using (SqlCommand expandCommand = new SqlCommand(expandQuery, tx.Connection, tx))
                            {
                                for (int i = 0; i < datas.Count; i++)
                                {
                                    expandCommand.Parameters.AddWithValue($"@param{i}", datas[i]);
                                }

                                expandCommand.ExecuteNonQuery();
                            }
                        }

                        if (sqlTransaction == null)
                        {
                            tx.Commit();
                        }
                        return newID;


                    }
                    catch
                    {
                        tx.Rollback();
                        throw;
                    }
                }
            }
            return null;
        }

        /// <summary>
        /// 构建SQL插入语句
        /// </summary>
        /// <param name="TableName">表名</param>
        /// <param name="Fields">字段列表</param>
        /// <param name="Datas">数据列表</param>
        /// <returns>返回SQL插入语句</returns>
        public string BuildInsertSQL(string TableName, List<string> Fields, List<string> Datas)
        {
            string FieldsString = string.Join(", ", Fields);
            string ValuesString = string.Join(", ", Datas.Select((data, index) => $"@param{index}"));

            string SQL =
                $"INSERT INTO [{TableName}] ({FieldsString})\n" +
                "OUTPUT inserted.PEB_ID as id -- 返回插入数据的ID\n" +
                $"VALUES({ValuesString});";

            return SQL;
        }
```

## 查询代码示例

同理，包括`SQL`语句构建 及 实现接口函数两部分

```csharp

        /// <summary>
        /// 执行查询并返回符合条件的数据集。
        /// </summary>
        /// <param name="whereString">SQL 查询中的 WHERE 子句。</param>
        /// <param name="groupBy">SQL 查询中的 GROUP BY 子句。</param>
        /// <param name="orderByField">用于排序的字段名称。</param>
        /// <param name="isAscending">是否升序排序。</param>
        /// <param name="pageSize">分页大小。</param>
        /// <param name="pageNumber">页码。</param>
        /// <returns>包含查询结果的数据集。</returns>
        public DataSet Select(string whereString, string groupBy, string orderByField, bool isAscending, int? pageSize, int? pageNumber)
        {
            string query = BuildSelectSQL(null, whereString, groupBy, orderByField, isAscending, pageSize, pageNumber);

            SqlConnection connection = GetSqlConnection();
            {
                OpenSqlConnection();
                using (SqlDataAdapter adapter = new SqlDataAdapter(query, connection))
                {
                    DataSet dataSet = new DataSet();
                    adapter.Fill(dataSet);
                    return dataSet;
                }
            }
        }

        /// <summary>
        /// 执行查询并返回符合条件的指定字段的数据集。
        /// </summary>
        /// <param name="fields">要查询的字段列表。</param>
        /// <param name="whereString">SQL 查询中的 WHERE 子句。</param>
        /// <param name="groupBy">SQL 查询中的 GROUP BY 子句。</param>
        /// <param name="orderByField">用于排序的字段名称。</param>
        /// <param name="isAscending">是否升序排序。</param>
        /// <param name="pageSize">分页大小。</param>
        /// <param name="pageNumber">页码。</param>
        /// <returns>包含查询结果的数据集。</returns>
        public DataSet Select(List<string> fields, string whereString, string groupBy, string orderByField, bool isAscending, int? pageSize, int? pageNumber)
        {
            string query = BuildSelectSQL(fields, whereString, groupBy, orderByField, isAscending, pageSize, pageNumber);

            SqlConnection connection = GetSqlConnection();
            {
                OpenSqlConnection();
                using (SqlDataAdapter adapter = new SqlDataAdapter(query, connection))
                {
                    DataSet dataSet = new DataSet();
                    adapter.Fill(dataSet);
                    return dataSet;
                }
            }
        }

        /// <summary>
        /// 执行查询并返回符合条件的数据类对象。
        /// </summary>
        /// <param name="whereString">SQL 查询中的 WHERE 子句。</param>
        /// <returns>查询到的数据类对象。</returns>
        public object SelectReturnObject(string whereString)
        {
            string query = BuildSelectSQL(null, whereString);

            SqlConnection connection = GetSqlConnection();
            {
                OpenSqlConnection();
                using (SqlCommand command = new SqlCommand(query, connection))
                {
                    using (SqlDataReader reader = command.ExecuteReader())
                    {
                        if (reader.Read())
                        {
                            PeopleData peopleData = new PeopleData
                            {
                                Base = new PeopleBase
                                {
                                    PEB_ID = reader.IsDBNull(reader.GetOrdinal("PEB_ID")) ? 0 : reader.GetInt32(reader.GetOrdinal("PEB_ID")),
                                    PEB_Name = reader.IsDBNull(reader.GetOrdinal("PEB_Name")) ? String.Empty : reader.GetString(reader.GetOrdinal("PEB_Name")),
                                    PEB_Sex = reader.IsDBNull(reader.GetOrdinal("PEB_Sex")) ? String.Empty : reader.GetString(reader.GetOrdinal("PEB_Sex")),
                                    PEB_Birthday = reader.IsDBNull(reader.GetOrdinal("PEB_Birthday")) ? String.Empty : reader.GetDateTime(reader.GetOrdinal("PEB_Birthday")).ToString("yyyy-MM-dd"),
                                    PEB_Job = reader.IsDBNull(reader.GetOrdinal("PEB_Job")) ? String.Empty : reader.GetString(reader.GetOrdinal("PEB_Job")),
                                    PEB_Title = reader.IsDBNull(reader.GetOrdinal("PEB_Title")) ? String.Empty : reader.GetString(reader.GetOrdinal("PEB_Title")),
                                    PEB_Employer = reader.IsDBNull(reader.GetOrdinal("PEB_Employer")) ? String.Empty : reader.GetString(reader.GetOrdinal("PEB_Employer"))
                                },
                                PrincipalExpand = new PeoplePrincipalExpand
                                {
                                    PEB_ID = reader.IsDBNull(reader.GetOrdinal("PEB_ID")) ? 0 : reader.GetInt32(reader.GetOrdinal("PEB_ID")),
                                    PE_Major = reader.IsDBNull(reader.GetOrdinal("PE_Major")) ? String.Empty : reader.GetString(reader.GetOrdinal("PE_Major")),
                                    PE_Speciality = reader.IsDBNull(reader.GetOrdinal("PE_Speciality")) ? String.Empty : reader.GetString(reader.GetOrdinal("PE_Speciality")),
                                    PE_Engage = reader.IsDBNull(reader.GetOrdinal("PE_Engage")) ? String.Empty : reader.GetString(reader.GetOrdinal("PE_Engage")),
                                    PE_Address = reader.IsDBNull(reader.GetOrdinal("PE_Address")) ? String.Empty : reader.GetString(reader.GetOrdinal("PE_Address")),
                                    PE_IsYouth = reader.IsDBNull(reader.GetOrdinal("PE_IsYouth")) ? String.Empty : reader.GetString(reader.GetOrdinal("PE_IsYouth")),
                                    PE_OfficePhone = reader.IsDBNull(reader.GetOrdinal("PE_OfficePhone")) ? String.Empty : reader.GetString(reader.GetOrdinal("PE_OfficePhone")),
                                    PE_MobilePhone = reader.IsDBNull(reader.GetOrdinal("PE_MobilePhone")) ? String.Empty : reader.GetString(reader.GetOrdinal("PE_MobilePhone")),
                                    PE_Email = reader.IsDBNull(reader.GetOrdinal("PE_Email")) ? String.Empty : reader.GetString(reader.GetOrdinal("PE_Email"))
                                }
                            };
                            return peopleData;
                        }
                    }
                }
            }

            return null;
        }

        /// <summary>
        /// 构建 SELECT SQL 语句。
        /// </summary>
        /// <param name="whereString">SQL 查询中的 WHERE 子句。</param>
        /// <param name="groupBy">SQL 查询中的 GROUP BY 子句。</param>
        /// <param name="orderByField">用于排序的字段名称。</param>
        /// <param name="isAscending">是否升序排序。</param>
        /// <param name="pageSize">分页大小。</param>
        /// <param name="pageNumber">页码。</param>
        /// <param name="fields">要查询的字段列表。</param>
        /// <returns>构建的 SELECT SQL 语句。</returns>
        private string BuildSelectSQL(List<string> fields, string whereString, string groupBy = null, string orderByField = null, bool isAscending = true, int? pageSize = null, int? pageNumber = null)
        {

            string fieldString = fields != null && fields.Count > 0 ? string.Join(", ", fields)
                : @"
            PeopleBase.PEB_ID ,
            PeopleBase.PEB_Name ,
            PeopleBase.PEB_Sex ,
            PEB_Birthday,
            PeopleBase.PEB_Job ,
            PeopleBase.PEB_Title ,
            PeopleBase.PEB_Employer ,

            PeoplePrincipalExpandData.PE_ID ,
            PeoplePrincipalExpandData.PE_Major ,
            PeoplePrincipalExpandData.PE_Speciality ,
            PeoplePrincipalExpandData.PE_Engage ,
            PeoplePrincipalExpandData.PE_Address ,
            PeoplePrincipalExpandData.PE_IsYouth ,
            PeoplePrincipalExpandData.PE_OfficePhone ,
            PeoplePrincipalExpandData.PE_MobilePhone ,
            PeoplePrincipalExpandData.PE_Email ";

            string query = $"SELECT {fieldString} FROM PeopleBase " +
                $"LEFT JOIN PeoplePrincipalExpandData ON PeopleBase.PEB_ID = PeoplePrincipalExpandData.PEB_ID";

            if (!string.IsNullOrEmpty(whereString))
            {
                query += $" WHERE {whereString}";
            }
            else query += " WHERE 1 = 1";

            if (!string.IsNullOrEmpty(groupBy))
            {
                query += $" GROUP BY {groupBy}";
            }

            if (!string.IsNullOrEmpty(orderByField))
            {
                query += $" ORDER BY {orderByField} {(isAscending ? "ASC" : "DESC")}";
            }

            if (pageSize.HasValue && pageNumber.HasValue)
            {
                int offset = (pageNumber.Value - 1) * pageSize.Value;
                query += $" OFFSET {offset} ROWS FETCH NEXT {pageSize.Value} ROWS ONLY";
            }

            return query;
        }
```

