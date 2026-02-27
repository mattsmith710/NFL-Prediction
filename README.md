::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {#quarto-document-content .content role="main"}
:::::::: {#title-block-header .quarto-title-block .default}
::: quarto-title
# Does Passing or Rushing Lead to More NFL Wins? {#does-passing-or-rushing-lead-to-more-nfl-wins .title}
:::

:::::: quarto-title-meta
::::: {}
::: quarto-title-meta-heading
Author
:::

::: quarto-title-meta-contents
Matt Smith
:::
:::::
::::::
::::::::

For my final project for Unstructured Data Analytics, I wanted to
enhance my knowledge on the NFL. I have always been an avid sports fan,
but the NFL has been the one of four major sports leagues in America
that I don't follow. Maybe that's becuase I am a DC sports fan and our
owner through my childhood cared more about money then a good product on
the field :).

I found statistics on each NFL team from the 2025 season on
sports-reference.com. These stats contain offensive and defensive
metrics for each game, and ultimately showed if the team won or lost to
go along with the stats each game.

As a casual NFL viewer, I've always heard people ask during Super Bowls,
"why don't they just throw it more?" While I do have enough knowledge
about football to understand why you can't throw it every single play,
it does bring up the question about if running or passing is more
important when it comes to the outcome of the game. Would you rather
establish the run or the pass game? I will be attempting to see if
there's statistical evidence to determine if the run or pass is more
important to winning games.

I first need to scrape the data from sports-reference.com. I notice the
pattern in the url where the only difference between each teams pages is
the three letter acryomn identifying each team. I create a list of the
different teams acryomns to loop over later. I also looked through the
column names and I know some would be converted to due to identical
title names. I decided to make a dictionary of column headers I wanted
to be a bit more clear, this will be put to use later after the
scraping.

::::: {#a4919bf3 .cell execution_count="1"}
:::: code-copy-outer-scaffold
``` {.sourceCode .python .code-with-copy}
import pandas as pd
import numpy as np
import random
import time

teams = [
    'crd', 'atl', 'rav', 'buf', 'car', 'chi', 'cin', 'cle', 'dal', 'den',
    'det', 'gnb', 'htx', 'clt', 'jax', 'kan', 'sdg', 'ram', 'rai', 'mia',
    'min', 'nwe', 'nor', 'nyg', 'nyj', 'phi', 'pit', 'sea', 'sfo', 'tam',
    'oti', 'was'
]

#Some weird names in the column headers
renamed_dict = {
    'Unnamed: 5':'Home', 'Rslt':'Win', 'Pts': 'Tm_PTS', 'PtsO': 'Opp_PTS',
    'Cmp': 'pCmp', 'Att':'pAtt', 'Cmp%': 'pCmp%', 'Yds': 'passYds', 'TD': 'pTD', 
    'Y/A': 'pY/A', 'AY/A': 'pAY/A', 'Rate':'pRate', 'Yds.1':'SaclYds', 'Att.1':'rAtt',
    'Yds.2':'rushYds', 'TD.1':'rTD', 'Y/A.1':'rY/A', 'Yds.3':'PntYds', 'Pass':'fdPass',
    'Rsh':'fdRush', 'Pen':'fdPen', 'Pen.1':'Pen', 'Yds.4':'PenYds'
}
```
::::
:::::

This indentifies I just want last year's data.

::::: {#ffe67af8 .cell execution_count="2"}
:::: code-copy-outer-scaffold
``` {.sourceCode .python .code-with-copy}
seasons = range(2025, 2026)
```
::::
:::::

This code runs the scraper. The first for loop will go over every season
in the range of seasons I identify, which in this case is just the one.
The second for loop will look at for every season all 32 teams and scrap
their statistics for each of their 17 regular season games. A team
dataframe will get created each iteration, and that will be concated to
the nfl dataframe, which is what we will be using for this project.

This code block was run in my Apple Computer's Terminal, due to issues
with VS Code

::::: {#5921d666 .cell execution_count="3"}
:::: code-copy-outer-scaffold
``` {.sourceCode .python .code-with-copy}
nfl_df = pd.DataFrame()
team = teams[0]
season = seasons[0]
import requests
from io import StringIO

cookie =  'is_live=true; osano_consentmanager_uuid=94d27b52-7498-458a-bcd1-7df7bed7cf1e; _ga=GA1.1.1647437913.1771899332; hubspotutk=058fec56e465da8d0f2ccf770d90ae4a; __hssrc=1; _li_dcdm_c=.pro-football-reference.com; _lc2_fpi=09239dd740e1--01kj6pwg974yze1ds2xfa0z8fy; _lc2_fpi_meta=%7B%22w%22%3A1771899339047%7D; cookie=07fd5b70-1339-46a8-abe7-4fa5c2c0b1b9; _lr_env_src_ats=false; _pubcid=2b87279c-ad1a-4412-b9a4-63e8ebba3144; gamera_user_id=a9dbfd6a-187e-49d3-ad48-dad1750370dc; _cc_id=36e42c134d6ec7b283fc18251204a2d6; panoramaId_expiry=1772504139233; panoramaId=88b9804db4121db4fb4543fb76c14945a7024066139ab5527b0db77b54948819; panoramaIdType=panoIndiv; pbjs_fabrickId=%7B%22fabrickId%22%3A%22E1%3AYU9ZfTO20MUf0yEkUnbRDQ4a3MZjIMPZwZ6BSD05kTnJjgFKHBg2W9rbVvzQdCh16DCQDGe6v16zlJoaykCL17YI0mhmpLAWgNmgmSPU2IuME2pGKtU1nvVry-nLvasSoxxBGLKjYCO5tGo5mhfCmA%22%7D; mm-user-id=Fq9oo1YUf1zCPBlP; _au_1d=AU1D-0100-001771899339-KH7WYM15-AC7S; ccuid=15101e7f-c43f-4228-9cc7-6ec331b8858f; gcid_first=06064d0b-9af3-4891-bc23-f9571014456e; _sharedID=2b87279c-ad1a-4412-b9a4-63e8ebba3144; _sharedID_cst=qCxDLBcsxQ%3D%3D; _lr_sampling_rate=100; TAPAD=%7B%22id%22%3A%220acc4154-b3a2-46d5-ab0a-cae7a0186e32%22%7D; sr_n=1%7CTue%2C%2024%20Feb%202026%2002%3A32%3A43%20GMT; pbjs_fabrickId_cst=rgwzbg%3D%3D; meta_more_button=1; _ga_FVWZ0RM4DH=GS2.1.s1771941509$o2$g1$t1771941760$j60$l0$h0; __cf_bm=5za07ySR3SJb35VyBdcK9LxcSjI0dCktuNuijW9irxc-1771956611-1.0.1.1-FvPhkQ2uihGecLJl0qJT31OPl9CmZ2K9zEdD_mfCLf3xD0Btl8St6uuZKvVxHfDil9dPdf_kyMZHvK9URAWa8GQT61QBlApbYwHqAbNj7q8; cf_clearance=7XSE62wjupoWZjBQt17oZqdtSV9Gg6u_f0R_k0C5BLM-1771956615-1.2.1.1-YTirG24R.MxxWTwVNw2qBlA_mRg9sj9QB2MteFfOHGSWtKqWEZvi8dlz982PNq8wGg6EKGtwjUW8x4k3cNYTtuCSM_KcJUyXrw269HCCqcPZ0VqlP5MKF2whImRDY9LFkG.M1pcoJK_a2PjtW0skZrhjSrydi54jPeuyoswM_E07waGxv6H7ky8cNmLPFzVxb_V_Fuu_s8w2EiFTS1vSVye1M1t4401SpJP0tkax3PA; srcssfull=yes; __hstc=223721476.058fec56e465da8d0f2ccf770d90ae4a.1771899332317.1771941528622.1771956616403.3; __hssc=223721476.1.1771956616403; _ga_EMBDG7RM0K=GS2.1.s1771956616$o3$g0$t1771956616$j60$l0$h0; _ga_80FRT7VJ60=GS2.1.s1771956616$o3$g0$t1771956616$j60$l0$h0; cookie_cst=znv0HA%3D%3D; connectId=%7B%22vmuid%22%3A%22Ue5mw_SH65l2sk7cx-fBFX_MIPHWdPYILrDrDUAmtN43Y8CasykVRjbKfBX9jwelsbz86xbpjf_P4XORVxE-8g%22%2C%22connectid%22%3A%22Ue5mw_SH65l2sk7cx-fBFX_MIPHWdPYILrDrDUAmtN43Y8CasykVRjbKfBX9jwelsbz86xbpjf_P4XORVxE-8g%22%2C%22connectId%22%3A%22Ue5mw_SH65l2sk7cx-fBFX_MIPHWdPYILrDrDUAmtN43Y8CasykVRjbKfBX9jwelsbz86xbpjf_P4XORVxE-8g%22%2C%22ttl%22%3A86400000%2C%22puid%22%3A%2260f955e8-ff7f-4d54-8cf8-eb26d1889a4c%22%2C%22lastSynced%22%3A1771899339183%2C%22lastUsed%22%3A1771956617151%7D; _lr_retry_request=true; mm-session-id=YFlMPgU7uF7ympio; gc_session_id=fcsmxcoauuffrwuxx6lyzi; __gads=ID=86d4ae15849645f3:T=1771899340:RT=1771956618:S=ALNI_MYiACxl7ZTBpjhGPERZBwJlSifgNw; __gpi=UID=00001397b4eef454:T=1771899340:RT=1771956618:S=ALNI_MY7iA0s7kUQtzxT8IpG2kgOY2ZUeA; __eoi=ID=6fdc20877483242b:T=1771899340:RT=1771956618:S=AA-AfjaDMImnT4mCJlPKL6QCz3iX; cto_bundle=p_FicV9DUThDcHppMUFsdmFadHBySURtJTJCdGk4VHhTYjZUVXNJNEZ4N3JPciUyQkhQeiUyQjE5Q1VhTk9UQWRLc0toY3J0d3NMN29YTG96b1BDZDgzZFhrSzdNTk1NbmNjVzFPQVJyY3VCS243JTJGJTJGTng3Y0Q3VUUxOU1iaXYzRnlBYkZrTGpoMnVtWUNHdXA4MXEydkRVVkR6JTJGeUZOamdxaiUyQkJ6UmNKdzZhalhUS01sdnkzUW1OeFdmR0MzTTVRZmtURG5NOFZwNU1KUVhhVW9KeXpCWmtIcXdkZnZEb0ElM0QlM0Q; cto_bidid=eGtRcl9ETFdPQ0VMRyUyRktndUUzTnAwU3poWXRzWlMyYmRXRUxISE10R09ma3dBalgzUEYlMkZjTXZKMjROWlZ1MmU5SW9UUHZmbHlNZ0ZPaEJ2d04yczBGS1lidWpwRkNYTURpRDZvSnVzVEVyQkZqOGRJbjF6ZUdseldxRURGUkN6aERtcjMyM2dMcUlNTUd4cXRFb3E5VTE1UVlXdzRrUXdHQ2xhNHBjYWUlMkJGUDIzZ00lM0Q'

headers = {
    "Referer": "https://www.pro-football-reference.com/teams/crd/2015/gamelog/?__cf_chl_tk=hxnpyIIRWKZhCg.G6cQ0o.yxBuSOpiVB91MOLLh3TQE-1771956611-1.0.1.1-w0hka1CMQOOUOQNEJTSqdh9WJtP2F7EJAu.Wm9KjeT8", 
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36", 
    "Cookie": cookie    
}

payload = "3b9e902f3b1b2003fb3825cb274ba67a5a9dbbbc9f7bc01a576a82752d02d54f=UPbs0irsry5UGMXA476Cbpk0JHINz2CTL7tZONvK3PI-1771956611-1.2.1.1-7GSBcqcxSyJuv2Xj7xQczTwRQlDs.Lth9fOdVYbwuypqP.Kr3jz4OmlQ29FPkiIuMmOkEL3DKMoWOx.9B2pUYkxt4YDF7TbYg7B4xxnZiWTGkHkKsnN3oIrFSM.uOq.jbJaS3Yrs5QP1_qIxnvpO_gZyRZFCsEpB_s5LZjJrBCGzg_fzWQ_5K3DZTGLtbPslwB1pWvlmi1eZojZRypr42GqFKHjeZWweaKhlK7IP3OTtinga1uDqME7.LkqPpXiUPnOZr6uAZoUE9Xp6hklYuCYd2Zeukf47KS.rHLwGIB6VS7C7L5Xh5l91wn76y8b1w_LdLw9p1QlmQXXWuxUA8mVNM_8RTvkBK.nPP_yukNgFMD5eoX4bFnkVh7M462oE2EZ.626H_m9NJ7RoKwpEeo_AXgzoN_otxySDFnNc5W1S5oQ8LeeAXa.rkI1rk9s1hbufUqHgWf4LJwNmQLkALBQm67rU6ByejPg7tD0GHUsZup2Q0hynHN1hWRUEHHFYYxi5urobTNta3UoHhhR4T86c9KG6in8As7APK95bFRTlHAkdqefZjrjAkKTbi3wy_MCJ.RcICzFN2CsIpKb0uz0bhkVBxUcVJlqrrhKfkmIRLzrcPW4L5au2mhAOO5l55ulh6yGZxNDRXNSbbD7ycnhWrql4y69.V4mONt7SnY2WmWWFA88HTCQ6MG_2OyNQ2QCM74IrWdvuPxOGkvPS6AlTx_4GWfwZlE3Ym58Z7Y0GhnSF7a9cOERoEs3CZIaOL.Z5GKvksmdD1At40c6s.gxJFsJ4e6030vaTw9sefmyXHB.EnyyzYSRNcgkk4ynBv6zahtfvgJxgoLKewjQKSEspJZIcTK6JT0A.zbf5ruFAptYSEInhmSmJAaYz0BQhem8oySfRVaeOj5pPIA4W5YOHmIAlv9fJPtmXH3KcSEnJoq0Y_tJvbJNM4X8LdcHhqPVZzryki2_USc7sbzne4Dbnh4OIliBjMWpbMSDOzOpCcsccQyCzgB1fQZ2geDU9_8Y2TDrSOWDn02IwpbJtsBJcgDL.WNLKpv8vjtZnA1DXoKRCUBfvFJhqnmmOlggonb3DcyMbwwvTcNCj1lsc7tbuDho4LbEx1VJ5WpdYihVFxKO56TEvmbfMsttV72.X&dde87d71df0a1dde00d441d49d98d4b5849933fe7c277fcaba918ec19f794b6c=qxswBB9P2qvtCqQaB91B5rjHCe_rZlysDGO2a_4wTyQ-1771956611-1.2.1.1-uWODbrVSnTrkOwYrSu2XjsYeqpq7YnKj28X9RpyO_4bwzqu.WSyADBJcl5bpLi1EsjeJ8XJhWG6tJimzU.Fz1iL5h9bTkgOAt2da7Wq2KaD6RjKBc6deYIOkLF0hDGmUptvlKuL8eI6BJFHCC0FqgMMZmvQL9TzDNajIGeXN33moidf.1aAIO7jX9v1otisdQkHchvCkzdx2obpDGR2u_8h3_m3d95MtR2E_sn57.I8AXKuyp17WjfXLifirX5ibJEr3u8Ec5vdhJdx_iRHM8F0ejUXr_zPZZdcg_hDYUQ02t6Eq.k3Slz9hp8YHI29GZTKgxN_pfxbun.C6.W9FFSuc0BOCIT1z1N_r2MzF.zBSORlEuAigN._4VPcN1L6rrNx5AL7YEDowP2.AdE1J89_GH.l5ndvloa7pb4H2nfvvZF__kS_VLQsVn9E0vPUr.5hrPFzsSSUbSU6bLFolxcHkCSN1jPDruzy5EDyAJGvYZst.iiZMgZAlvjXEREpPj40J7xWJDEPBr3jpDg4l7a.ua51ksIkX3KpqNJbrtVcWLugeUnYNqOZDeh9gcsYRtRY_o2.1MU9LjF8t.hLbZKHW4x.XResZaQIwquelp678Ot2VdZLufgV3Vn7tsH5ZV7TeROOi6YFXTOLmRsTPF9YVlgEkdWhsLTYWQ0mrlKGXEcj4rHGF9hpdZZYitvDXHzO5IaTQ8ANiFCxnA_sfwumW4vUUMJaOPS5x3qkZjzceRfzB6pJsk5u0AMshP3e506fwiXcZx_wEemyr3cYPGbaLtgPe0IxHebBgnU3CyWwTaKJ3Jc5ki6LA4yIinEWhAbMjWFq1ldr1sD383nkh9QbHUQGk2eXuAgla_3obDipKWp7rY7E6qlhqwUffr9jpFhGPakR2hErBEbyR42nTIcErpNp26DZ0zx8LyHynLqUjsjCkf8_Kygaw4AgFQMsMnj1qCo9S2mpV_mQDcxWnGiuNuTLYkFlWhYnUruLdXymkICVErsuawmlxV0czBV5Uu61RHQ55urav5ZhOPgK.8PWvW50ztineTi5TbC6qUjyqVtV.fKnht_lfP5ZtYRJeI77uucAfq55lZtvvnJmUr2fsnO2c.y7td40gdduQwZ21bWJUVPQP.5r01WpsSREqrb.b.LgOEaUFgHk6pvoStjqdJKfcZVKyBxvtNuN_2cVC_9f57Qb.tOaumgzvbYr6Vyp2BAoQGSq5kHfnZ2d5DUCgwDJ4EC7KiX.7dA9dJNjbgirnQQEt1t1uN.eI1E0ROH6kmTzvfQVAzP.H_huqRJa4BWr0IsYISlKUIRNF4n8KPNEIf8jkTeGf0D.o7VPI.JLBy1zPR20Q3MXBC5zuDtzE8W0h5XNNN4lx78T2Ag24cXYQ6UkPzpPeb6_RN9qDIt3gKm2YPzf4hg5AxLqV3HQK_WWE0dDIz6x4CAyi1ukUN5gqXAg7A9XgWGD87nvi152u2FixGXIvXYW27WH4x5GXWzPU6vtkVNvKMHUcC8sYXX8wrOuACfa5BO6R2ZxKpjo81MVfkUJXxxEpiFZ6lVuS5cRYNTl0XwUxiIE5Zg_vNHoIcNvoKeJ7BuG49Rh8nudZQMJtJqtuZlN0vuT7.AQjkgeQkYxCyMc5bxV2o1YWngrUOvrCq7PPke5pCOvUXeM9YkPvAbrcVTrVjMZzsSybSFKd9985_xg7EFLmcMtP5TMraaEVcNxLbJIe92Kqq.tIWWku9EReF7y4NERxX5dAjqhHe5GrRT4Dkz8alNW.6zqDWyAJgzEl_BpGk8JMTUZ523ov9SCWFQS1a7Y0LwYEI6vJI5x01.hF8ALLhRYwcYKqVsZpYb1gh39BSzSd.UFEKnzeyeUvCAo5W8c_RwGJPRh8uhAHsIxtT95CAS4GFynSiXITbkO1HfUe2WnZXmvQzCj6vmUpu9BkNY3o3RUe5EI17NnTASMXAwbpOmA6ICROpAlPoIsgSdCW8mOVRvHc7G.SPHh7nksjzvPyF0vPJGZOOJtv1namKJ_XkHEJnHadvhOuTIKGyhgjJzfS_P2BaauvwFB58mXDluJtDk8YQUtI80I8Qx_lHHhBv.q0NIjWNxDXdOJHUmmOin7EK7QH.YCJ8bxVb0tt.YNnB1SBSz0DzVulPrs8w4hn.qzTBw__89pJ_taV1ve0lciFub2ospKPsiK.UIryKGJVNnpfWZfG7i4afgCN0..GLzIcmn3pITOxXaX8V6GkFTYxO6ZhZiRwXnoLLC80na3i8HZH8IyAXP3FbLp35umSYVab1Tmw8Ey0gWVtj5lFaJamw0v4F1d.rWVnYTG2oSoII58n1IaJ4z1Pf6sVvFLJ1z6JF7pfhZ_uH7RbyJELFGoVsjmxvn1jExvhflBEVmsaJ0DC08wfSsobH5kqRsFh34OPn.UHpBmUueApIEppjRbFfIItHQBtG1HoJnsGdW.bTmBlCjTZEMNRuLrYlkn1bDkZQueHcS2qkrtFOh4UmCyXoneTG.2Pm8oQ1_DrdIc8VHwL8DzU7CYWouajS946_GzFi83lhNoTeDRj9KOWaa7yXoPyKab8U6bXHybmFLmMw3TTk1Lbr0Uq0kYqLlsBcP2_VSd15.Q.k68l0YzwwL58KKSN.TsBp9.fTwultSvmALxxC50Muwt7hOJVb3fpCD1FYk60sLqRGn0BJzNCmDOfK9XSaC_QXm02l_PKk5uPQuWeANl6MH1ssl0UuempHZcwwAVRBW6zkqCHWy.f9SWwAfeOm3e7.SCV8OzUdoIWuvdAMI7VmP1aGeajgrqN_werpTV12gDm8A_kEcBDRh2BPpXP_msGkvK5tt6HvONXkpiTTpy7KsSzABA2HhRaYH_k.YxvxnXn9M0RBfEk60dLpzzexWw.ClgugSthivhlHYcTP7n61Ee4VI7wiWPjkh4Mtj._dLfw0c264GLNxAY2Um.lMBymmMp2RZptKpbQ7MXLRrNhNce8cixGNm7fOADO4LCoREmbH6GcuZmksd_6UOq6A9_Dx8bHt9NSfIZxgcj.XEAd.uGTPeQANyi51pu7xJp_7mrgex1pbWwdJTxuGNNoI_Jga48QfLdPohRwnNcqICgB7URNj1mwBBKjw__pcK.3OXy.shNZyB0BeU9fKSetSuZVyNEupfx314vtZB.9k7VI5gR9XmDWpj1YrE7i_TrMqROApeIFBicubNaniBUfMD1nIduCc6oBl7TfK.GVlHGsaKwKBiNmAZWbtEkEobiShiwmsK5Cma3GKE9SJg0R8ZoRFw_yTuXMocpd_T23pP5AvuNj1ckb_pJgln7CAj2b8F24qCSgtj1cpXMGJz9..JsM83K2EA2xiwBA40xwj6A9Oy9KeaVDZ24NFufUydKWXV3FppwhTpWHt9FTBE9hOZkPUg9thm6dRoKJ4v.0cejDA_Iq_N6bhqd7klm3mL8wnaoOYYhQ9f_8Nfb1c6GjYFR.iVHFFaNFT_645o6su6b2OyY8JYu9opuoppSNznyP.HhLbbHfUcvIRbKipn.ntG9Mh83SzAMRuSos99WkWXn1YN531b.ZS50K6R6l5Ktvn3qK6TP.T4CnS25PKw9dbe8Ntg_mxnBDCVR.8mFNVBVXjrsTsvV8nsXpyCMwc0eDzIWwSfiwG8It5kqXeyEoGnPKd1r9qfmwWM2XgVeb2IsBurpHSbh2wA7QXS8T9znXksnD2e825yxPht6NGxCth.Kjk7rt0_8_fb1ntSf0apj515otSQHpY6wT.N3DzS_H9MzzjcNWf0fOQybh98_b2aN1ygDvQBa1ONqXF90LI.KJPxilJ8FCeWFXQovOH2s7EqHvBpWAaf.JW2HsHZlaYYpLYnPgHVKIqGNYAOBKsEwixKz8cZLBGdKnuuTUjmTlSbDQfjseGw9CuF2BhpEd_gff3QiHcWbDovtWYwGSS9Dg_Vu9Bv6JbA5ZCj6HvqDZXBi08.S.mW259RlV8d3WKdFI_dJoSSB1BPuzSRU76CqpHp8KLWchJ6QfATwJ2_qR6GxCnLwuTB_R.BWcWO74Mf8k09Xz4.qMsz5N0AZpbJU.AZiN9zG0d56sO3zX2S8llRrEeeLfWkdBvP.h6iqOQA_udwzkse35bjWdSQdn1QDBqfbi_G8QSOJFYhZnuE_OWOJqwzQvZ7mlNsH1Y_fNa_7jJHk88YSVku7shr0JWW5ziatN6HayoKF_PREFnY_tZscAEvFI68Vrrc9Vc8BVi8BLxsD_RjDroFeHd7d8MuN1IpERZAXUeNDRHPmRdmjEZUdYUErawNhrvJqQyofHGwyus9ZoR9RKoJPAZl.GsfZK3lruKaEOGtUrdGzm7F2_ilO60rtCzTNt4t3vjYYNjan_ostiHRUtT_vEthvvjZ2dfP_fKPH017FOkDyJbFWfVVMOJTX881dq99EAHR2X0T6rc_PIoMOmKcyUAW.dSqa19Ec7z.4PBPiqgiUnV6XwmMBAFIteh74Kp0.5AOoTf8cObdHoSQjAkGmRu82aHkHMfwhUqYgkh1thnuMLL_2jS7oYQCjEGlMFi6.lN.VIrEenogGL2Sv.BPkswpifyj4v3U70NoLH.CzHB1.TgvWAyZeSVfq2hTap4PinmzHoGcmxPY__UCIWd_XITZdFccjSVOUZXJCn0OnVFwIQvAxtZVH6Es0V6l2R0Z.F0zC7HUNXo_SruI_4e5VIpQunz8YjdTIzBKWJaOqA9HO.HR9PlLgbmDKhjtqpDDZ7H1456MmNqvorijdlbx4qHUrdWNfIOVsA3i8j1vsXqZQKNx22wbxPKAb.JWfs8dTtmYqeTVIUSU6KduYx.r8onYex5UaP.d5wuOjLKhULji748oD_tIOzCCYbZh0HvYXYOPWngNIuWYVVd0HUiz5Al41mdgrkLMqwzLj1qs101SFd3Dr_B8A1dl6G_w.NuTfmoQVRzt.6lHpEiWcXEHPUphCNvDxzNzeB2oT8fYi34v8YFuwySRQ1dKpljG7eqqEfeBoJHgih8iOGszV7xm5dRTX7jXxwvVo419qhesfvrt.R5NXdGu2j2jN3InukYbvivKoz0fVVwgoYupwITdslFS35.J4vVn8zQ7Zap2xDyJpNBXeMfUBsrfKRUYQ2sBqLslbhH4wMVOkANeb56GrjW8qw2XzhO0Ar0e026uii2q8S3cHFniWSREnqB5tui8.Y6w7WbRJLoDImsrdq00z1Ag1qP11CdETPwfqet1YHZBSCuIBgKkUdfBnACMRwuPHn6ybJ5XzPaqAkbbT72wGzFS81f86amo1IgL8FBJKX5so5IDO5eKx3wIQTycXlSXIpnthrJzaOwfKytCXFj.IA3d2HYEXvk3XuO8_OAChJPvs2yEyje9kyXMMlJtGJml1WI7NbVFhcUj5cKKlnI3vyFCOuBa98JnOnwO.NERbLIN0OzGu0xH.FkHdE9SYwpAPviMwopdFVj1REvAx3gvqPgjTxmMI1KGUwTrKN5Eil.DGMfFBszjLfGW_nD4pCnSW5ySZvPXdLsJakXA78BOj.NlqFPV1qj_DJRYfLDLJAUj7MZAnW6DVBtY5NomIOj1aNLwGvgC9KTViRvbVtPu5Kana02UPnAvS0Ufr_pr5rusoBYcV69twec5y4mHn2RPa6fQmNzuQn8miHMHrJ0osmZDk6c2KG3v7DxT_ROyX.YFo64N8QB0dAluBwxV6rRgFEOoeU2bO5mq..x8KyLlrUJ1r8qTQP2R7xUsQt3HyZoPZCBFdyW4v7nhF1GyJO5hT.wrrND8cmju2G1S9XyYZhds2zQ7hPW5thHruB_obDkWeTCNiu8Ri7eOpkjVYmVRRd4yJq5U_nqmiAOThccMqXIMfH0SYq975B.Zw97pbdRQsj_FK29xKZKhj.JUX.470DqN5x74MGB3RuN6oFZq6_eJhPi7FXWIwHhwfolwy3Kz6EmwUbj_.yVUVa2NrBOqYbGPmYVZbAemQrwCi0PIp3s.cswuZRq2RC8H4QQH6NFaWI2ruJnHoRY68AXgfv404UgaesqyiR6wW_szO_OomarH5shcmzI5WRFLqrvXPiETGixaKNq4Pp51IycRaDXlDwYbhbkdCZCi4FEGUJILEG8N8bvbWeoFdIeBFfizwNq2ESYQGQNHxSxInOCWfl6JSwBV6WFQWwR.VIPIxY3IdhmlpaZQGrFN07qFb9mXuhP6KcUe5l4BBkWOO9CfM1_WwgINTYexZNTcSCigzJ1h40caou_cw9JjRe3mM42R4hnE3Zz.K0j3eb077d53wq.yHhkDUKiBZPKP5ZpHZZ7WW935ATRVXXb1SEYg._Di5QOmxvmNdjW9yF.yT5opfAGvlnXEuL6MYjpK6_eMSaW2hRg3zVU0dyqbYjo29E5C2DWC2N4rJ_bvLaxPXMfzA8GA20mzNlAGsMHG7aYT8F0hl6aJTt..OBU2zXiCbOMfxiShJ7eygE2IDMM.r8.fWWyC0bo5j_Cp43i1sQYnhf5bgPAfiaONmmQp7ddBEH7UR7oHeiuEZs7ML18X5nPdgcGG1ycOxXeT.h93gMAg3Rvsabf7it62SLTielPVQf8J8.yyViNxXQWK1VFJLbcKXAojXgB6BZ3RT7GGKEnPGxYtlXxOkgcua0xwKjqv92.7IcQwOME2CJqM1rquvoer_BEVjzViewR3LbFLxfN8Glenm.sTOOAxPmJ_zVYfDKbnQsbkHqW76Fg85vlMUh5Yu0mdLpiA_py6A_gCUxVqta2d1yan64opF9FHi97JWxSBCINnrp8KUYnqbOBiz0NuHoHQNODwVdRb4HwUdhos2dZFTPSiajcqkT7OZCf7hrgkgb_9ZC_DYTS2oDQ2B0h8GXoAZQDDkPWu89IyZMh1UDrabaoukdkzjJqk5asmQuO6C36hTju9H2cIPmjTYyqZeifY.TUw0tMcaS7iIaRjxCbtJK.t8We5qxDP6.iq0RJ2VdKtSmouQTBQGu84Iz872K9A0m_X..dJ9nqOdmSnIcTswbM4xIHs8elFQs.V8Ea2mL83SpcDXcVNy0Jj97y.CVRTID.13NZVBeogQJ8WJO3E9d6UZHf2IU0WuTne96_1CQ02IgxZpdkNDFR8z1I1FnPyyqvK3tW7ej87Naj5kvOAP7ggZQz5wMdVrBcBGyJi4cp9r7VsirS45IGM56RCiyhw.nMvwhiKB6aqgkHN8Um9CQNbC77olorvfVLb.6vQeTjQ33_kPhlzNWE.w.wFixntJjxzks1w8YCFphLeOCumHDi9dJK8stwM7Q9VDctcld766pura6NtLM8j3G720w_WiVBvnpgMQPRMRKfmM225N_zrxK2ImikVeoAtosfAzLQPKQDXcyKoycZM1_iwEVrx75bEL37WrRrkxzBhveldr46S83T_zwEJ0tygx_1QoWuheN6JZh47eTWRRu6JIGdIVDQLc1zm6kMBdF_9CdS0hj5AkzREICY5wwAAOD3uFFNnV91od0E6OHEoGCngBaZYAjR652FoznPQoKPllMUdEkq0qQMXtSbJZt_tCo8LvA2rM5aikEa1sB9vk4FPB42wSQaUGQ1H2HB9.sxC90x1iqH8qJCZQvYT44F96oD9DEygtv_H3JKzC.z91fbmADGcuXIiK8uxB7_jR9MUhjuraq7gD0hiOr5XAJHAlIYgUS8_iSnSdxngmd_wUVe2T0tyq42Ak2qA5vfh5mulDDxnSIOhTqq2akPTbgemhtc0HfyHE0yJ5TXOAvu49I73NjbTti8Jv4sVU2Rr3unmxidpo6Evb9tn8fR592vRVsys6db29T17XthNLT9qbrJeSz8.4XDemYEzpGsCmnECYmwWwklZiQz9ZUN9AFWOIVq79qW_E2BOT2DF5Pzi65LuLL0wCjJjmwXPnAd.2BdVGF2CUXooaV_5ZPt42jLOep5nOVr7jfJRmDXRHQutHVc7bRTsgJY2VDz6lvWZF9Tk4gWwC5YVCf9fQPsNVWXWc_2S6MyANONl_YShMtl6MKuJPJWElv32LTxDMON3G4TvOG7xylow&b1252cdf9b899e108a4c9a835e346d9ac90c61d8a578823684d2738bbc5fdc56=VOFe2wl9f2qS4UsUW5Fof95auF8i9DawsHWIah70Ku8-1771956615-1.0.1.1-heccF2KHcTDOmVe_7pQo9f_7IbMe7pI3o3AQMQKO_WRHZzy1aaHNfxxJiZpwfu5H_4Bz.wc0OKTpf2l9gW3KU8695tvir_UQmsv32NfnHYYGQvlMPneZSuwxMKn_3o_XLCIJ5lQEP2J1W7vVtcy9.dTXHRxNmy_6RX3sR1fmyIIYNKsaxoK57kc_juR6yB.u"

#Ran in terminal
for season in seasons:
    for team in teams:
        try: 
            time.sleep(random.randint(4,5))
            url = 'https://www.pro-football-reference.com/teams/' + team + '/' + str(season) + '/gamelog/'
            print(url)
            table_id = 'table_pfr_team-year_game-logs_team-year-regular-season-game-log'
            r = requests.post(url=url, json=payload, headers=headers)
            tm_df = pd.read_html(StringIO(r.text), header=1, attrs={'id':table_id})[0]
            nfl_df = pd.concat([nfl_df, tm_df], ignore_index=True)
        except:
            continue
```
::::
:::::

After running in the terminal, I typed this code in the terminal to make
the nfl_df a csv, to come back into VS Code.

::::: {#a69d9c88 .cell execution_count="4"}
:::: code-copy-outer-scaffold
``` {.sourceCode .python .code-with-copy}
nfl_df.to_csv('nfl_df.csv', index=False)
```
::::
:::::

This brings my csv back into VS Code.

::::::: {#c6a4f1ba .cell execution_count="5"}
:::: code-copy-outer-scaffold
``` {.sourceCode .python .code-with-copy}
nfl_df = pd.read_csv("~/Downloads/nfl_df.csv")
nfl_df.head()
```
::::

:::: {.cell-output .cell-output-display execution_count="3"}
::: {}
      Rk   Gtm   Week   Date      Day   Unnamed: 5   Opp   Rslt   Pts   PtsO   \...   3DConv   3DAtt   4DConv   4DAtt   Pen.1   Yds.4   FL   Int   TO   ToP
  --- ---- ----- ------ --------- ----- ------------ ----- ------ ----- ------ ------ -------- ------- -------- ------- ------- ------- ---- ----- ---- ----------
  0   1    1     1      9/7/25    Sun   @            NOR   W      20    13     \...   6        13      0        0       9       54      0    0     0    33:46:00
  1   2    2     2      9/14/25   Sun   NaN          CAR   W      27    22     \...   3        9       1        1       12      96      0    1     1    26:45:00
  2   3    3     3      9/21/25   Sun   @            SFO   L      15    16     \...   5        15      2        2       5       30      0    0     0    34:39:00
  3   4    4     4      9/25/25   Thu   NaN          SEA   L      20    23     \...   6        15      1        1       7       46      0    2     2    27:36:00
  4   5    5     5      10/5/25   Sun   NaN          TEN   L      21    22     \...   7        16      0        0       8       47      3    0     3    32:40:00

5 rows × 48 columns
:::
::::
:::::::

I will now change the column headers that I talked about above, as well
as make some new columns to make life a bit easier while doing
statistical analysis, as well as creating some columns on things I
believe could be interesting to look at.

:::::::: {#1c77de57 .cell execution_count="6"}
:::: code-copy-outer-scaffold
``` {.sourceCode .python .code-with-copy}
print(nfl_df.columns.tolist())
nfl_df = nfl_df.rename(columns=renamed_dict)
print(nfl_df.columns.tolist())


nfl_df['Win_Binary'] = (nfl_df['Win'] == 'W').astype(int)

nfl_df['Is_Home'] = (nfl_df['Home'] != '@').astype(int)

nfl_df['Margin'] = nfl_df['Tm_PTS'] - nfl_df['Opp_PTS']

#Won't be looking at ties
nfl_df = nfl_df[nfl_df['Win'] != 'T'].copy()

nfl_df.head()
```
::::

::: {.cell-output .cell-output-stdout}
    ['Rk', 'Gtm', 'Week', 'Date', 'Day', 'Unnamed: 5', 'Opp', 'Rslt', 'Pts', 'PtsO', 'OT', 'Cmp', 'Att', 'Cmp%', 'Yds', 'TD', 'Y/A', 'AY/A', 'Rate', 'Sk', 'Yds.1', 'Att.1', 'Yds.2', 'TD.1', 'Y/A.1', 'Ply', 'Tot', 'Y/P', 'FGA', 'FGM', 'XPA', 'XPM', 'Pnt', 'Yds.3', 'Pass', 'Rsh', 'Pen', '1stD', '3DConv', '3DAtt', '4DConv', '4DAtt', 'Pen.1', 'Yds.4', 'FL', 'Int', 'TO', 'ToP']
    ['Rk', 'Gtm', 'Week', 'Date', 'Day', 'Home', 'Opp', 'Win', 'Tm_PTS', 'Opp_PTS', 'OT', 'pCmp', 'pAtt', 'pCmp%', 'passYds', 'pTD', 'pY/A', 'pAY/A', 'pRate', 'Sk', 'SaclYds', 'rAtt', 'rushYds', 'rTD', 'rY/A', 'Ply', 'Tot', 'Y/P', 'FGA', 'FGM', 'XPA', 'XPM', 'Pnt', 'PntYds', 'fdPass', 'fdRush', 'fdPen', '1stD', '3DConv', '3DAtt', '4DConv', '4DAtt', 'Pen', 'PenYds', 'FL', 'Int', 'TO', 'ToP']
:::

:::: {.cell-output .cell-output-display execution_count="4"}
::: {}
      Rk   Gtm   Week   Date      Day   Home   Opp   Win   Tm_PTS   Opp_PTS   \...   4DAtt   Pen   PenYds   FL   Int   TO   ToP        Win_Binary   Is_Home   Margin
  --- ---- ----- ------ --------- ----- ------ ----- ----- -------- --------- ------ ------- ----- -------- ---- ----- ---- ---------- ------------ --------- --------
  0   1    1     1      9/7/25    Sun   @      NOR   W     20       13        \...   0       9     54       0    0     0    33:46:00   1            0         7
  1   2    2     2      9/14/25   Sun   NaN    CAR   W     27       22        \...   1       12    96       0    1     1    26:45:00   1            1         5
  2   3    3     3      9/21/25   Sun   @      SFO   L     15       16        \...   2       5     30       0    0     0    34:39:00   0            0         -1
  3   4    4     4      9/25/25   Thu   NaN    SEA   L     20       23        \...   1       7     46       0    2     2    27:36:00   0            1         -3
  4   5    5     5      10/5/25   Sun   NaN    TEN   L     21       22        \...   0       8     47       3    0     3    32:40:00   0            1         -1

5 rows × 51 columns
:::
::::
::::::::

I first wanted to compare the correlation of some categories surrounding
passing and throwing to winning the game. This was a good start to see
which stats to dive into deeper.

:::::::: {#b9cf3747 .cell execution_count="7"}
:::: code-copy-outer-scaffold
``` {.sourceCode .python .code-with-copy}
from scipy import stats
import matplotlib.pyplot as plt
cols_to_check = ['pRate', 'passYds', 'pCmp%', 'rAtt', 'rushYds', 'rY/A', 'Sk']

for col in cols_to_check:
    nfl_df[col] = pd.to_numeric(nfl_df[col], errors='coerce')

for col in cols_to_check:
    r, p = stats.pointbiserialr(nfl_df['Win_Binary'], nfl_df[col])
    print(f"{col}: r={r:.3f}, p={p:.4f}")


# store results
results = {}
for col in cols_to_check:
    r, p = stats.pointbiserialr(nfl_df['Win_Binary'], nfl_df[col])
    results[col] = r

# cleaner labels for the chart
labels = {
    'pRate':    'Passer Rating',
    'passYds':  'Pass Yards',
    'pCmp%':    'Completion %',
    'rAtt':     'Rush Attempts',
    'rushYds':  'Rush Yards',
    'rY/A':     'Rush Y/A',
    'Sk':       'Sacks Taken'
}

# sort by correlation value
sorted_results = dict(sorted(results.items(), key=lambda x: x[1]))
bar_labels = [labels[k] for k in sorted_results.keys()]
bar_values = list(sorted_results.values())
bar_colors = ['green' if v > 0 else 'red' for v in bar_values]

# plot
plt.figure(figsize=(10, 6))
plt.barh(bar_labels, bar_values, color=bar_colors, alpha=0.7)
plt.xlabel('Correlation with Winning (r)')
plt.title('Which Stats Correlate Most with Winning?')
plt.tight_layout()
plt.show()
```
::::

::: {.cell-output .cell-output-stdout}
    pRate: r=0.442, p=0.0000
    passYds: r=0.135, p=0.0016
    pCmp%: r=0.255, p=0.0000
    rAtt: r=0.510, p=0.0000
    rushYds: r=0.352, p=0.0000
    rY/A: r=0.033, p=0.4364
    Sk: r=-0.243, p=0.0000
:::

:::: {.cell-output .cell-output-display}
::: {}
<figure class="figure">
<p><img src="Unstructured_vF2_files/figure-html/cell-8-output-2.png"
class="figure-img" width="950" height="566" /></p>
</figure>
:::
::::
::::::::

It seems that the amount you rush has the highest correlation to
winning, followed by the QB's passer rating.

After running correlations, we know which stats relate to winning but
not how useful they actually are for predicting it, which is what AUC
answers.

We run StandardScaler first because yards go from 0-500 while completion
% goes from 0-100, and without scaling the model treats those
differences as equal which isn't fair.

::::::::: {#69a43cd3 .cell execution_count="8"}
:::: code-copy-outer-scaffold
``` {.sourceCode .python .code-with-copy}
from sklearn.linear_model import LogisticRegression
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import roc_auc_score

scaler = StandardScaler()

for col in ['pRate', 'passYds', 'pCmp%', 'rAtt', 'rushYds', 'rY/A', 'Sk']:
    X = scaler.fit_transform(nfl_df[[col]])
    lr = LogisticRegression()
    lr.fit(X, nfl_df['Win_Binary'])
    auc = roc_auc_score(nfl_df['Win_Binary'], lr.predict_proba(X)[:,1])
    print(f"{col}: AUC={auc:.3f}")

results = {}
for col in cols_to_check:
    X = scaler.fit_transform(nfl_df[[col]])
    lr = LogisticRegression()
    lr.fit(X, nfl_df['Win_Binary'])
    auc = roc_auc_score(nfl_df['Win_Binary'], lr.predict_proba(X)[:,1])
    results[col] = auc

plt.figure(figsize=(10, 6))
plt.barh(list(results.keys()), list(results.values()))
plt.axvline(0.5, color='red')
plt.xlabel('AUC Score')
plt.title('Which Stats Best Predict Winning?')
plt.legend()
plt.show()
```
::::

::: {.cell-output .cell-output-stdout}
    pRate: AUC=0.754
    passYds: AUC=0.583
    pCmp%: AUC=0.642
    rAtt: AUC=0.802
    rushYds: AUC=0.705
    rY/A: AUC=0.521
    Sk: AUC=0.635
:::

::: {.cell-output .cell-output-stderr}
    /var/folders/np/7fjdm1cn2610_myq2cjs84gw0000gn/T/ipykernel_89881/2416066872.py:27: UserWarning: No artists with labels found to put in legend.  Note that artists whose label start with an underscore are ignored when legend() is called with no argument.
      plt.legend()
:::

:::: {.cell-output .cell-output-display}
::: {}
<figure class="figure">
<p><img src="Unstructured_vF2_files/figure-html/cell-9-output-3.png"
class="figure-img" width="826" height="523" /></p>
</figure>
:::
::::
:::::::::

I put a red line at 0.5, which is a coin flip worth of a chance.
Anything near there is useless for our evaluation. As we saw before
rushing attempts has the highest AUC value, followed by passing rating
of the Quarterback.

This initially made me think that rushing the ball was the more
important part of the offense, until I thought about it a little bit
more. Every time a team is up big in the fourth quarter, I see them run
the ball on the majority of plays in order to burn the clock out. So
finally I will look to see if what the relationship between scoring
margin and rush attempts is to try to answer the question, "Do good
teams run the ball more, or do good teams run the ball more because
their leading?"

:::::::: {#6eb215e7 .cell execution_count="9"}
:::: code-copy-outer-scaffold
``` {.sourceCode .python .code-with-copy}
import matplotlib.pyplot as plt

plt.figure(figsize=(10, 6))
plt.scatter(nfl_df['Margin'], nfl_df['rAtt'], alpha=0.5)
plt.xlabel('Score Margin')
plt.ylabel('Rush Attempts')
plt.title('Score Margin vs Rush Attempts')
plt.legend()
plt.show()
```
::::

::: {.cell-output .cell-output-stderr}
    /var/folders/np/7fjdm1cn2610_myq2cjs84gw0000gn/T/ipykernel_89881/1352179348.py:8: UserWarning: No artists with labels found to put in legend.  Note that artists whose label start with an underscore are ignored when legend() is called with no argument.
      plt.legend()
:::

:::: {.cell-output .cell-output-display}
::: {}
<figure class="figure">
<p><img src="Unstructured_vF2_files/figure-html/cell-10-output-2.png"
class="figure-img" width="808" height="523" /></p>
</figure>
:::
::::
::::::::

As we can see, there's a strong positive relationship between scoring
margin and rush attempts. This means the more that you are up, the more
rushing attempts you have.

Now lets look at the relationship with passing attempts and margin, to
get a comparision.

:::::::: {#6b3c1039 .cell execution_count="10"}
:::: code-copy-outer-scaffold
``` {.sourceCode .python .code-with-copy}
import matplotlib.pyplot as plt

plt.figure(figsize=(10, 6))
plt.scatter(nfl_df['Margin'], nfl_df['pAtt'], alpha=0.5)
plt.xlabel('Score Margin')
plt.ylabel('Pass Attempts')
plt.title('Score Margin vs Pass Attempts')
plt.legend()
plt.show()
```
::::

::: {.cell-output .cell-output-stderr}
    /var/folders/np/7fjdm1cn2610_myq2cjs84gw0000gn/T/ipykernel_89881/3890930136.py:8: UserWarning: No artists with labels found to put in legend.  Note that artists whose label start with an underscore are ignored when legend() is called with no argument.
      plt.legend()
:::

:::: {.cell-output .cell-output-display}
::: {}
<figure class="figure">
<p><img src="Unstructured_vF2_files/figure-html/cell-11-output-2.png"
class="figure-img" width="808" height="523" /></p>
</figure>
:::
::::
::::::::

There seems to be little to no relationship between passing attemtps and
scoring margin. I would expect to see a negative relationship due to
teams being down wanting to throw the ball down the field, taking more
risks. This does not seem to be the case.

The last thing I wanted to look at was to look at only close games, to
get rid of the games when one team is running the ball up big.

:::::::: {#bc320176 .cell execution_count="11"}
:::: code-copy-outer-scaffold
``` {.sourceCode .python .code-with-copy}
from sklearn.linear_model import LogisticRegression
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import roc_auc_score

close_games = nfl_df[(nfl_df['Margin'] >= -10) & (nfl_df['Margin'] <= 10)]

scaler = StandardScaler()
cols_to_check = ['pRate', 'passYds', 'pCmp%', 'rAtt', 'rushYds', 'rY/A', 'Sk']

results = {}
for col in cols_to_check:
    X = scaler.fit_transform(close_games[[col]])
    lr = LogisticRegression()
    lr.fit(X, close_games['Win_Binary'])
    auc = roc_auc_score(close_games['Win_Binary'], lr.predict_proba(X)[:,1])
    results[col] = auc

import matplotlib.pyplot as plt
plt.figure(figsize=(10, 6))
plt.barh(list(results.keys()), list(results.values()))
plt.xlabel('AUC Score')
plt.title('Which Stats Best Predict Winning in Close Games?')
plt.legend()
plt.show()
```
::::

::: {.cell-output .cell-output-stderr}
    /var/folders/np/7fjdm1cn2610_myq2cjs84gw0000gn/T/ipykernel_89881/2343458209.py:23: UserWarning: No artists with labels found to put in legend.  Note that artists whose label start with an underscore are ignored when legend() is called with no argument.
      plt.legend()
:::

:::: {.cell-output .cell-output-display}
::: {}
<figure class="figure">
<p><img src="Unstructured_vF2_files/figure-html/cell-12-output-2.png"
class="figure-img" width="826" height="523" /></p>
</figure>
:::
::::
::::::::

The top two attributes seem to be consistent, but a balance offense
still seems to be the best strategy.

There seems to be pretty conclusive evidence that both passing and
rushing are needed as a balanced offense keeps their opponents on their
toes. Looking at the passing game, the data clearly shows that
efficiency matters more than volume. Passer rating was a stronger
predictor of winning than raw passing yards, and completion percentage
beat out passing yards as well. This tells us that teams that win aren't
just throwing the ball more, they're throwing it more accurate. Sacks
also stood out as a meaningful signal, with win rate dropping sharply as
sacks allowed went up, suggesting that offensive line play has a real
and measurable impact on outcomes.

The rushing game told the most interesting story in the whole analysis.
Rush attempts appeared to be the single strongest predictor of winning,
but the scatter plot exposed why that number is misleading. Teams that
are already winning simply run more to burn clock, meaning rushing
volume is a consequence of the score rather than a cause of winning. To
hammer that point home, rush yards per carry had almost no relationship
with winning at all, sitting at an AUC of just 0.521. The takeaway is
that running efficiently doesn't win you games, and running a lot just
means you were probably already ahead.
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
