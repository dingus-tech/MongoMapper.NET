## MongoMapper.NET

A .NET Document Mapper for MongoDB over MongoDB C# Driver

### Dependencies

* [MongoDB distributions] (http://www.mongodb.org/downloads)
* [MongoDB Driver for CSharp] (http://github.com/mongodb/mongo-csharp-driver)
* [NUnit] (http://www.nunit.org), [NUnit for Visual Studio] (http://nunitforvs.codeplex.com)

### Defining the Model

		[Serializable]
		[MongoKey(KeyFields = "Code")]    
		[MongoCollectionName(Name="Paises")]
		[MongoGeo2DIndex(IndexField="Pos")]
		[MongoGeo2DSphereIndex(IndexField="Area")]
		[MongoRelation("PersonRelation","Code","Person","Country")]
		public class Country : MongoMapper<Country>
		{				  
			#region Public Properties
			
			[BsonElement("c")]
			public string Code { get; set; }

			[BsonElement("n")]
			public string Name { get; set; }

			[BsonElement("p")]
			public long[] Pos { get; set; }

			public GeoArea Area { get; set; }

			#endregion

			#region Public Methods

			[MongoPropertyValidator(PropertyName = "Code")]
			public void CodeIsUp(string CountryCode)
			{
				if (CountryCode != CountryCode.ToUpper())
				{
					throw new Exception(String.Format("{0} must be {1}", CountryCode, CountryCode.ToUpper()));
				}
			}

			#endregion
		}
				
		public class CountryCollection: MongoMapperCollection<Country> {}
	
### Work with the Model

		var c = new Country {Code = "es", Name = "España"};
		try
		{
			c.Save();
			Assert.Fail();
		}
		catch (ValidatePropertyException ex)
		{
			Assert.AreEqual(ex.GetBaseException().GetType(), typeof (ValidatePropertyException));
			c.Code = "ES";
			c.Save();
		}

		c = new Country {Code = "UK", Name = "Reino Unido"};
		c.Save();

		c = new Country {Code = "UK", Name = "Reino Unido"};
		try
		{
			c.Save();
			Assert.Fail();
		}
		catch (DuplicateKeyException ex)
		{
			Assert.AreEqual(ex.GetBaseException().GetType(), typeof (DuplicateKeyException));
		}

		using (var t = new MongoMapperTransaction())
		{
			var c2 = new Country {Code = "US", Name = "Francia"};
			c2.OnBeforeInsert += (s, e) => { ((Country) s).Name = "Estados Unidos"; };
			c2.Save();

			t.Commit();
		}

		var c3 = new Country();
		c3.FillByKey("US");
		Assert.AreEqual(c3.Name, "Estados Unidos");

		if (!c3.IsLastVersion())
			c3.FillFromLastVersion();

		var countries = new CountryCollection();
		countries.Find();
		Assert.AreEqual(countries.Count, 3);

		countries.Find().Limit(2).Sort(countries.Sort.Ascending(C=>C.Name));
		Assert.AreEqual(countries.Count, 2);
		Assert.AreEqual(countries.Total, 3);

		countries.Find(countries.Filter.Or(MongoQuery<Country>.Eq(co => co.Code, "ES"), MongoQuery<Country>.Eq(co => co.Code, "UK")));
		Assert.AreEqual(countries.Count, 2);


### You can find examples in the Test Project 

* [Class Definition](https://github.com/emiliotorrens/MongoMapper.NET/tree/master/EtoolTech.MongoDB.Mapper.Test.NUnit/Classes) 
* [MongoMapperCollection] (https://github.com/emiliotorrens/MongoMapper.NET/blob/master/EtoolTech.MongoDB.Mapper.Test.NUnit/MongoMapperCollectionTest.cs)
* [Find] (https://github.com/emiliotorrens/MongoMapper.NET/blob/master/EtoolTech.MongoDB.Mapper.Test.NUnit/FindTest.cs)
* [Write] (https://github.com/emiliotorrens/MongoMapper.NET/blob/master/EtoolTech.MongoDB.Mapper.Test.NUnit/InsertModifyDeleteTest.cs)
* [Incremental ID] (https://github.com/emiliotorrens/MongoMapper.NET/blob/master/EtoolTech.MongoDB.Mapper.Test.NUnit/IncrementalIdTest.cs)
* [Relations](https://github.com/emiliotorrens/MongoMapper.NET/blob/master/EtoolTech.MongoDB.Mapper.Test.NUnit/RelationsTest.cs) 
* [Events] (https://github.com/emiliotorrens/MongoMapper.NET/blob/master/EtoolTech.MongoDB.Mapper.Test.NUnit/EventsTest.cs)
* [Extension Methods] (https://github.com/emiliotorrens/MongoMapper.NET/blob/master/EtoolTech.MongoDB.Mapper.Test.NUnit/ExtensionTest.cs)
* [Original Version] (https://github.com/emiliotorrens/MongoMapper.NET/blob/master/EtoolTech.MongoDB.Mapper.Test.NUnit/OriginalObjectTest.cs)
* [Memory Transactions] (https://github.com/emiliotorrens/MongoMapper.NET/blob/master/EtoolTech.MongoDB.Mapper.Test.NUnit/TransactionTest.cs)
* [Document Version] (https://github.com/emiliotorrens/MongoMapper.NET/blob/master/EtoolTech.MongoDB.Mapper.Test.NUnit/VersionTest.cs)
 


