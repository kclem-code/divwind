;*************************************************
; wind_1.ncl
;*************************************************
load "$NCARG_ROOT/lib/ncarg/nclscripts/csm/gsn_code.ncl"   
load "$NCARG_ROOT/lib/ncarg/nclscripts/csm/gsn_csm.ncl"    
load "$NCARG_ROOT/lib/ncarg/nclscripts/csm/contributed.ncl" 
;*************************************************
begin
;*************************************************
; open file and read in data: data are on a Gaussian grid
;*************************************************

  ufile = addfile ("uwnd.nc", "r")
  u1=short2flt(ufile->u)
  vfile = addfile ("vwnd.nc", "r")
  v1=short2flt(vfile->v)

  printVarSummary(u1)
  printVarSummary(v1)

  time     = u1&time
  lev = u1&level
  lat = u1&latitude
  lon = u1&longitude

  ntim  = dimsizes(time)                 ; get dimension sizes  
  nlev  = dimsizes(lev)                                               
  nlat  = dimsizes(lat)  
  nlon  = dimsizes(lon)      

  print(ntim)
  print(nlev)
  print(nlat)
  print(nlon)
  print(time)
  print(lev)
  print(lat)
  print(lon)

  print(u1(0,0,:,20))
  print(v1(0,0,:,20))

  uvmsg = 1e+36
  udiv1    = new ( (/ntim,nlev,nlat,nlon/), float, uvmsg )  ; zonal divergent wind
  vdiv1    = new ( (/ntim,nlev,nlat,nlon/), float, uvmsg )  ; meridional divergent wind

  system ("/bin/rm udivwnd.nc")         ; remove any pre-existing file
  system ("/bin/rm vdivwnd.nc")         ; remove any pre-existing file

  udivo    = addfile("udivwnd.nc","c" )  ; zonal divergent wind
  vdivo    = addfile("vdivwnd.nc","c" )  ; meridional divergent wind

  udiv1@long_name  = "Zonal Divergent Wind"
  vdiv1@long_name  = "Meridional Divergent Wind"
  udiv1@units      = u1@units
  vdiv1@units      = v1@units
  copy_VarCoords(u1,udiv1)
  copy_VarCoords(v1,vdiv1)

;*************************************************
; calculate divergence: Use Wrap to include meta data
;*************************************************

  div = uv2dv_cfd (u1,v1,u1&latitude,u1&longitude, 3) ; get divergence

;*************************************************
; calculate divergent wind components 
;*************************************************    

  dv2uvg(div,udiv1,vdiv1) ; div  ==> divergent wind components
  

   udivo->udiv1 = udiv1                          ; 4D               
   vdivo->vdiv1 = vdiv1

end
