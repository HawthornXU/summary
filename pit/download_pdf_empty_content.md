https://stackoverflow.com/questions/51981473/downloaded-pdf-looks-empty-although-it-contains-some-data
       
       1. 
       // const xhr = new XMLHttpRequest();
        // xhr.open("get", `${document.location.origin}/fr-api/sellers/${shortId}/qr/pdf`, true);
        // xhr.responseType = 'blob'
        // xhr.setRequestHeader('auth', campaignToken);
        //
        // xhr.onload = () => {
        //     // const res = xhr.response;
        //     // file.saveBlobResponse(xhr.response);
        //         const url = window.URL.createObjectURL(xhr.response);
        //         const a = document.createElement('a');
        //         a.style.display = 'none';
        //         a.href = url;
        //         a.download = '1.pdf';
        //         document.body.appendChild(a);
        //         a.click();
        //         window.URL.revokeObjectURL(url);
        // }
        // xhr.send();
        // const file = new File([data], seller.sellerName, {type: "application/pdf"});
        
        2. 
        const res = await fetch();
        return res.blob();
        
        
        
