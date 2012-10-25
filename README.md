elasticjavademo
===============
For Creating Indexes
=====================


	
         private static final String JDBC_DRIVER = "com.microsoft.sqlserver.jdbc.SQLServerDriver";
	private static final String CONNECTION_URL = "jdbc:sqlserver://ip:1433;databaseName=xyz";
	private static final String USER_NAME = "";
	private static final String PASSWORD = "";
	private static final String QUERY = "query";
	static  String indexName = "index";
	static  String indexType = "type";

public static void main(String args[]) {
 		try{
 			Class.forName(JDBC_DRIVER).newInstance();  
			Connection conn = DriverManager.getConnection(CONNECTION_URL, USER_NAME, PASSWORD);  
			String sql = QUERY;  
			Statement stmt = conn.createStatement();  
		         ResultSet rs = stmt.executeQuery(sql);
		    
		    Client client = NodeBuilder.nodeBuilder().local(true).settings(ImmutableSettings.settingsBuilder()
	 	    		.put("cluster.name", "abc")).node().client();
		
	 	    int i=1;
			 while(rs.next()){
					System.out.println(i);
					client.prepareIndex(indexName, indexType, String.valueOf(i)).setSource(XContentFactory.jsonBuilder().startObject()
		 			 .field("columnName1",rs.getString("ColumnName1"))
		 			 .field("columnName2",rs.getString("ColumnName2"))
		 			 )
		 	        .execute()
		 	        .actionGet(); 
					i++;
				}
			 client.admin().indices().prepareFlush().execute().actionGet();
		   }
 		 catch (Exception e)
 	    {
 	      e.printStackTrace();
 	    }
}



For Searching query:
============================

	
public static void main(String args[]) {
 		try{
 			
 			 Client client = NodeBuilder.nodeBuilder().local(true).settings(ImmutableSettings.settingsBuilder()
 					.put("cluster.name", "abc")).node().client();

 			 client.admin().cluster().health(new ClusterHealthRequest("companygroupindex").waitForYellowStatus()).actionGet();

 			 QueryBuilder queryBuilder = QueryBuilders.termQuery("columnName1", "query");
 	 	     SearchRequestBuilder searchRequestBuilder = client.prepareSearch("index");
 	 	     searchRequestBuilder.setTypes("type");
 	 	     searchRequestBuilder.setSearchType(SearchType.DEFAULT);
 	 	     searchRequestBuilder.setQuery(queryBuilder);
 	 	     searchRequestBuilder.setFrom(0).setSize(60).setExplain(true);
 	 	     SearchResponse resp = searchRequestBuilder.execute().actionGet();
 	 	     
 	 	     for (SearchHit hit : resp.getHits()){
 	 	     System.out.println("Hit ID: "+hit.getId());
 	 	     }
			
 	    }
 	    catch (Exception e)
 	    {
 	      e.printStackTrace();
 	    }
 }
