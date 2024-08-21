MariaDB standartlarında yazıldı,
1054 satır.

Id, CityDistricts, ParentId adlı 3 sütundan oluşmaktadır.

İl sütunları => ParentId NULL
İlçe Sütunları ParentId => 1 - 81 Id sahip il sütunlarına sahip.



.NET csv kullanım =>

 public class CitiesDistricts
 {
     public int Id { get; set; }
     public string CityDistricts { get; set; }
     public int? ParentId { get; set; }
     public CitiesDistricts Parent { get; set; }
     public ICollection<CitiesDistricts> Children { get; set; }

     public CitiesDistricts(int id, string cityDistricts, int? parentId = null)
     {
         Id = id;
         CityDistricts = cityDistricts;
         ParentId = parentId;
         Children = new List<CitiesDistricts>();
     }
 }

-------------------------------------


public class CitiesDistrictsMap : IEntityTypeConfiguration<CitiesDistricts>
{
    public void Configure(EntityTypeBuilder<CitiesDistricts> builder)
    {
        builder.HasOne(e => e.Parent)
            .WithMany(e => e.Children)
            .HasForeignKey(e => e.ParentId)
            .OnDelete(DeleteBehavior.Restrict);

        var citiesDistrictsList = new List<CitiesDistricts>();

        var csvLines = File.ReadAllLines("CSV/CitiesDistricts.csv");

        foreach (var line in csvLines.Skip(1))
        {
            var columns = line.Split(';');

            int? parentId = string.Equals(columns[2], "NULL", StringComparison.OrdinalIgnoreCase)
                ? (int?)null
                : int.Parse(columns[2], CultureInfo.InvariantCulture);

            var cityDistrict = new CitiesDistricts(
                id: int.Parse(columns[0], CultureInfo.InvariantCulture),
                cityDistricts: columns[1],
                parentId: parentId
            );

            citiesDistrictsList.Add(cityDistrict);
        }

        builder.HasData(citiesDistrictsList.ToArray());
    }
}


-------------------------------------

public DbSet<CitiesDistricts> CitiesDistricts { get; set; }

-------------------------------------
