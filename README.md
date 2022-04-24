# GoogleMaps-Integration-Angular
How to integrate google map with making marker cluster  in Angular dynamically.


Install googlemaps using below command,
 npm install --save @types/googlemaps

Before Integrating Make sure buy a ApiKey from google cloud platform and add in your index.html file,
 <script src="http://maps.googleapis.com/maps/api/js?key=APIKEY"></script>
 APIKEY = your apikwy
  
  
If you want to add heat map layer your script will be like below,
  <script src="http://maps.googleapis.com/maps/api/js?key=APIKEY&libraries=visualization&callback=initMap"></script>

Rendering value from base component to map component using button click,
basecomponent.html,
    <button  class="okcl btn btn-secondary down"
               (click)="btnclick()">MapView</button> 

     <div id="comp-render" *ngIf="display">
       <app-map [gmapList]="gmapList"></app-map>
    </div>
baseComponent.ts,
 @Output() gmapList:any[] = [] 
  //gmapList - your input data,..
  
Mapcomponent.html,
<div #map style="width:100%;height:600px"></div>

Mapcomponent.ts,

 map: any;
  heatmap: google.maps.visualization.HeatmapLayer; 
  @ViewChild('map', {static:true}) mapElement:any;
  @Input() gmapList=[];
  gmpDupList:any[]=[];
  markerArray:any[]=[];
  current_value = [];
  prev_value=[];
  latitude = 43.879078;
  longitude = -103.4615581;
 
    
   constructor( private apiService: ApiService ) {  }

    ngOnChanges(changes:SimpleChanges): void{
     for (let propName in changes) {
        let chng = changes[propName];
        this.current_value  = chng.currentValue;
        this.prev_value = chng.previousValue;
      }
      if (this.prev_value == undefined){
        console.log("inside if", this.gmpDupList)
        this.gmpDupList = this.current_value
        this.GmapChanges()
      }
      else{
        this.gmpDupList = []
        this.markerArray =[]
        this.gmpDupList = this.current_value
        this.GmapChanges()
      }
    }

     GmapChanges(){ 
 
    
    const mappy = this.gmpDupList.map(({emp_id,punch_type,punch_time,latitude, longitude}) => 
    ({emp_id,punch_type,punch_time,latitude, longitude}));
   
    const mapProperties = {
      center: new google.maps.LatLng(this.latitude, this.longitude),
      zoom: 2,
      mapTypeId: google.maps.MapTypeId.ROADMAP
      
 };
 console.log("map element",typeof(this.mapElement))
 this.map = new google.maps.Map(this.mapElement.nativeElement, mapProperties);

//adding multiple markers which has same lat and long
 var marker_icon = {
  url:"http://maps.google.com/mapfiles/ms/icons/purple-dot.png",
  labelOrigin: new google.maps.Point(10, 11)
};


 mappy.forEach(location => {
  var marker = new google.maps.Marker({
    position: new google.maps.LatLng(location.latitude, location.longitude),
    title:location.emp_id + '-'+ location.punch_type+'-'+ location.punch_time,
    map: this.map,
    icon: marker_icon,
    
  });  
    this.markerArray.push(marker);
  });

  // adding cluster to our map
  // const test = new MarkerClusterer(this.map, this.markerArray,
  //   {imagePath: 'https://developers.google.com/maps/documentation/javascript/examples/markerclusterer/m'}     
  // );
  
  const heat_data = this.gmpDupList.map(({latitude, longitude}) => ({latitude, longitude}));
  console.log("heatmapdata", this.gmpDupList)
  var heatmapData = this.gmpDupList.map(function(item) {
  return new google.maps.LatLng(item.latitude, item.longitude)
  });
  var heatmap = new google.maps.visualization.HeatmapLayer({ 
  data: heatmapData, 
  radius:20,
  maxIntensity: 2,
  dissipating: true,
  });
  heatmap.setMap(this.map)

  }

}
